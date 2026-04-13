# OFR Issue Tracker — Migration & Installation Guide

**Version:** 1.0
**Package Date:** February 2025
**Estimated Deployment Time:** ~2 hours

---

## What's in This Package

The OFR Issue Tracker is a fully M365-native risk issue management system built with SharePoint Online, Power Apps, and Power Automate. This package contains everything needed to deploy the solution into a new M365 environment from scratch.

### Solution Components

| Component | Technology | Description |
|-----------|-----------|-------------|
| **3 SharePoint Lists** | SharePoint Online | OFR_Issues (tracker), OFR_UpdateHistory (audit trail), OFR_IntakeQueue (triage queue) |
| **1 Power Apps Canvas App** | Power Apps | 6 screens: Dashboard, Tracker, Issue Detail, Submit, Group Allocation, Kanban — with 2 side panels |
| **2 Power Automate Flows** | Power Automate | Daily Staleness Calculator (scheduled) + Intake Promotion (instant/triggered) |
| **Documentation** | Markdown + HTML | SDD, test plan, user guide, tear sheet, build guides |

### Architecture

```
                    Microsoft 365 Tenant
 ┌──────────────────────────────────────────────────┐
 │                                                  │
 │   Entra ID ──► Power Apps ◄──► SharePoint Online │
 │   (SSO)        Canvas App      3 Lists           │
 │                    │                 ▲            │
 │                    ▼                 │            │
 │              Power Automate ─────────┘            │
 │              2 Cloud Flows                        │
 └──────────────────────────────────────────────────┘
```

**Zero infrastructure required.** No Azure subscriptions, no custom domains, no CI/CD pipelines. Everything runs within M365.

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **M365 Tenant** | Microsoft 365 Business Standard or above |
| **SharePoint Online** | Included with M365 — must be able to create sites |
| **Power Apps** | Power Apps Developer Plan (free) or Power Apps per-user license |
| **Power Automate** | Free tier included with M365 (sufficient for this solution) |
| **Admin Account** | Must have: SharePoint site creation + Power Platform environment maker permissions |
| **Browser** | Microsoft Edge or Google Chrome (for Power Apps Studio) |

---

## Quick-Start Deployment Checklist

Complete these steps in order. Each step references a detailed guide document.

```
[ ] Step 0: Fill in your environment values
             → 06-Environment-Config/ENVIRONMENT-VARIABLES.md

[ ] Step 1: Run find-and-replace on documentation (optional)
             → 06-Environment-Config/find-and-replace-checklist.md

[ ] Step 2: Create SharePoint site and 3 lists
             → 01-SharePoint/CREATE-SITE.md
             → 01-SharePoint/*.json (schema references)

[ ] Step 3: Load sample data (optional, recommended for testing)
             → 01-SharePoint/sample-data/*.csv

[ ] Step 4: Build Power Automate Flow 1 — Daily Staleness Calculator
             → 02-PowerAutomate/OFR-Daily-Staleness-Calculator.md
             → 02-PowerAutomate/flow-expressions/*.txt (copy-paste expressions)

[ ] Step 5: Build Power Automate Flow 2 — Intake Promotion
             → 02-PowerAutomate/OFR-Intake-Promotion.md
             → 02-PowerAutomate/flow-expressions/promotion-ItemID.txt

[ ] Step 6: Build Power Apps Canvas App
             → 03-PowerApps/REBUILD-GUIDE.md (overview + Intake Review panel)
             → 03-PowerApps/OFR-PowerApps-Completion-Guide.md (full build guide)

[ ] Step 7: Share the app with your M365 group
             → See Step 7 in 03-PowerApps/REBUILD-GUIDE.md

[ ] Step 8: Run the test plan
             → 04-Documentation/OFR-Test-Plan.md

[ ] Step 9: Optional — Pin app as a Microsoft Teams tab
             → See Step 8 in 03-PowerApps/REBUILD-GUIDE.md
```

---

## Deployment Order (Dependency Map)

Components must be created in this order due to dependencies:

```
Step 0: Environment Variables
    │
    ▼
Step 2: SharePoint Site
    │
    ├──► OFR_Issues list
    ├──► OFR_UpdateHistory list
    └──► OFR_IntakeQueue list
            │
            ▼
    Step 3: Sample Data (optional)
            │
            ▼
Step 4: Staleness Calculator Flow ─────┐
Step 5: Intake Promotion Flow ─────────┤
                                       │
                                       ▼
                              Step 6: Power Apps Canvas App
                                       │
                                       ▼
                              Step 7: Share App
                                       │
                                       ▼
                              Step 8: Test
                                       │
                                       ▼
                              Step 9: Teams Tab (optional)
```

**Key dependency:** Power Apps requires both the SharePoint lists (for data sources) and the Intake Promotion flow (for the Promote button) to exist before the app can be fully configured.

---

## Detailed Step Instructions

### Step 0: Configure Environment Variables

1. Open `06-Environment-Config/ENVIRONMENT-VARIABLES.md`
2. Fill in every row in the "Your Value" column:
   - `[TENANT]` — your tenant prefix (e.g., `contoso`)
   - `[TENANT-DOMAIN]` — your primary domain (e.g., `contoso.com`)
   - `[ADMIN-EMAIL]` — your admin email (e.g., `admin@contoso.com`)
   - `[ADMIN-DISPLAY-NAME]` — display name for sample data
3. Keep this file open — you'll reference it throughout the deployment

### Step 1: Find-and-Replace (Optional)

If you want the documentation files to reflect your environment:

1. Open `06-Environment-Config/find-and-replace-checklist.md`
2. Follow the file-by-file replacement instructions
3. Or use the bulk `sed` commands provided in the checklist

> **Note:** The files in `01-SharePoint/` and `02-PowerAutomate/` use `[TENANT]` as a placeholder. Replace it manually as you read each guide, or do a bulk replacement first.

### Step 2: Create SharePoint Site & Lists

**Time: ~20 minutes**

Follow `01-SharePoint/CREATE-SITE.md` to:
1. Create the "OFR Issue Tracker" team site
2. Create 3 lists with exact column definitions
3. Verify all columns and choice values

Reference the `*.json` schema files for precise column specifications.

### Step 3: Load Sample Data (Optional)

**Time: ~10 minutes**

Import the CSV files from `01-SharePoint/sample-data/`:
- 8 issues with varied priorities, statuses, and staleness levels
- 17 update history entries
- 2 pending intake items for testing

### Step 4: Build Staleness Calculator Flow

**Time: ~15 minutes**

Follow `02-PowerAutomate/OFR-Daily-Staleness-Calculator.md` step by step.

**Critical:** Use the exact expressions from `flow-expressions/` files — copy-paste to avoid transcription errors.

Test the flow manually after building.

### Step 5: Build Intake Promotion Flow

**Time: ~20 minutes**

Follow `02-PowerAutomate/OFR-Intake-Promotion.md` step by step.

This flow has more steps (7 actions) and requires careful dynamic content mapping. Test with a sample intake item ID.

### Step 6: Build Power Apps Canvas App

**Time: ~60-90 minutes** (the largest component)

1. Start with `03-PowerApps/REBUILD-GUIDE.md` for the high-level overview
2. Follow `03-PowerApps/OFR-PowerApps-Completion-Guide.md` for screen-by-screen control placement
3. The Intake Review panel formulas are in the REBUILD-GUIDE

### Steps 7-9: Share, Test, Teams

Follow the instructions at the end of `03-PowerApps/REBUILD-GUIDE.md` for sharing, testing, and Teams integration.

---

## Post-Deployment Verification

After all components are deployed, verify end-to-end:

| Test | Expected Result |
|------|-----------------|
| Open Power App | Dashboard loads with KPI cards showing correct counts |
| Click intake item | Review panel opens with Title, Description, Priority, Date |
| Accept intake item | New issue created in OFR_Issues, intake marked "Accepted" |
| Reject intake item | Intake item marked "Rejected", disappears from gallery |
| Navigate to Tracker | All issues display with correct staleness colors |
| Filter by "High" | Only high-priority issues shown |
| Search for an owner name | Matching issues displayed |
| Click issue row | Detail screen shows header + update history |
| Add an update | New entry in OFR_UpdateHistory, LastUpdated refreshes |
| Submit new issue | New item in OFR_IntakeQueue with "Pending" status |
| Run staleness flow | DaysSinceUpdate recalculates, StalenessFlag updates |

For comprehensive testing, use the 76-case test plan: `04-Documentation/OFR-Test-Plan.md`

---

## Troubleshooting

### SharePoint Issues

| Issue | Solution |
|-------|----------|
| "Access denied" when creating site | Ensure your account has SharePoint Admin or Site Creation permissions |
| Choice column values not saving | Verify choice values are spelled exactly as specified (case-sensitive) |
| CSV import columns don't match | Column names must match exactly — check for trailing spaces |

### Power Automate Issues

| Issue | Solution |
|-------|----------|
| Expression validation errors | Ensure you're pasting into the **Expression** tab, not the Dynamic content tab |
| "Item not found" on Get items | Verify Site Address URL is correct — no trailing slash |
| Staleness flow shows errors for some items | Check that all OFR_Issues rows have a value in LastUpdated |
| Flow doesn't appear in Power Apps | Go to Action → Power Automate → Add flow in Power Apps Studio |

### Power Apps Issues

| Issue | Solution |
|-------|----------|
| Data source not found | Re-add SharePoint connection: Data → Add data → SharePoint → re-enter site URL |
| Choice column Patch errors | Use `{Value: "text"}` syntax for all Choice columns in Patch formulas |
| Flow trigger errors | Verify flow name matches exactly in the formula. Re-add flow via Action → Power Automate |
| "Delegation warning" on Filter | Expected for complex filters — acceptable for lists under 500 items |
| Gallery shows no items | Check that the SharePoint list has data and the filter formula matches column names |

### General Issues

| Issue | Solution |
|-------|----------|
| Wrong environment | Check the environment picker in top-right of make.powerapps.com and make.powerautomate.com |
| Connection errors | Ensure the admin account has an active M365 license and SharePoint access |
| Flow permissions for users | Set Run-only access on both flows and share the SharePoint connection |

---

## Package Contents

```
OFR-Migration-Package/
│
├── README-INSTALL.md                          ← You are here
│
├── 01-SharePoint/
│   ├── CREATE-SITE.md                         ← Site + list creation guide
│   ├── OFR_Issues-schema.json                 ← 11 columns
│   ├── OFR_UpdateHistory-schema.json          ← 6 columns
│   ├── OFR_IntakeQueue-schema.json            ← 7 columns
│   └── sample-data/
│       ├── OFR_Issues-sample.csv              ← 8 records
│       ├── OFR_UpdateHistory-sample.csv       ← 17 records
│       └── OFR_IntakeQueue-sample.csv         ← 2 records
│
├── 02-PowerAutomate/
│   ├── OFR-Daily-Staleness-Calculator.md      ← Scheduled flow rebuild guide
│   ├── OFR-Intake-Promotion.md                ← Instant flow rebuild guide
│   └── flow-expressions/
│       ├── staleness-DaysSinceUpdate.txt       ← Copy-paste expression
│       ├── staleness-StalenessFlag.txt         ← Copy-paste expression
│       └── promotion-ItemID.txt                ← Copy-paste expression
│
├── 03-PowerApps/
│   ├── REBUILD-GUIDE.md                        ← Overview + Intake Review panel
│   └── OFR-PowerApps-Completion-Guide.md       ← Full 87KB construction guide (6 screens)
│
├── 04-Documentation/
│   ├── OFR-SDD.md                              ← System Design Document (neutralized)
│   ├── OFR-Completion-Guide.md                 ← Build summary (neutralized)
│   ├── OFR-Test-Plan.md                        ← 76-case test plan
│   ├── OFR-User-Guide.md                       ← End-user documentation
│   ├── OFR-Tear-Sheet.html                     ← Product tear sheet
│   └── OFR-Appkit4-Colour-Map.md               ← Appkit4 colour reference
│
├── 05-Reference-Docs/
│   ├── SDD_OFR_Issue_Tracker_M365.html
│   ├── SDD_One_Firm_Risk_Tracker.html
│   ├── BRD_One_Firm_Risk_Tracker.html
│   ├── Implementation_Playbook_OFR_Risk_Tracker.html
│   ├── Exec_Outline_One_Firm_Risk_Tracker.html
│   ├── Artifact_5_Incremental_Risk_Review_Summary_M365.html
│   ├── Solution_Overview_One_Firm_Risk_Tracker.docx
│   ├── User_Guide_One_Firm_Risk_Tracker.docx
│   └── User_Guide_One_Firm_Risk_Tracker.html
│
└── 06-Environment-Config/
    ├── ENVIRONMENT-VARIABLES.md                 ← Fill in your values first
    └── find-and-replace-checklist.md            ← Substitution checklist
```

---

## Support

| Resource | Location |
|----------|----------|
| System Design Document | `04-Documentation/OFR-SDD.md` |
| Test Plan | `04-Documentation/OFR-Test-Plan.md` |
| User Guide | `04-Documentation/OFR-User-Guide.md` |
| Product Overview | `04-Documentation/OFR-Tear-Sheet.html` |
