# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The OFR Issue Tracker is an M365-native risk issue management system built entirely with SharePoint Online, Power Apps, and Power Automate. **There is no traditional codebase** — no source files, no build system, no tests to run. This repository is a **migration/deployment package** containing documentation, schemas, and configuration needed to rebuild the solution from scratch in a new M365 tenant.

## Architecture

```
Entra ID (SSO) ──► Power Apps Canvas App ◄──► SharePoint Online (3 Lists)
                        │                           ▲
                        ▼                           │
                   Power Automate (2 Cloud Flows) ──┘
```

**Three-layer design:**
- **Data Layer:** 3 SharePoint lists — `OFR_Issues` (tracker), `OFR_UpdateHistory` (audit trail), `OFR_IntakeQueue` (triage queue)
- **UI Layer:** Power Apps canvas app with 6 screens (Dashboard, Tracker, Issue Detail, Submit, Group Allocation, Kanban) + 2 side panels
- **Automation Layer:** 2 Power Automate flows — Daily Staleness Calculator (scheduled) and Intake Promotion (instant)

## Repository Structure

| Directory | Purpose |
|-----------|---------|
| `01-SharePoint/` | Site/list creation guide, JSON schemas for all 3 lists, sample CSV data |
| `02-PowerAutomate/` | Step-by-step flow rebuild guides + copy-paste expression files in `flow-expressions/` |
| `03-PowerApps/` | Canvas app rebuild guides (REBUILD-GUIDE.md for overview, OFR-PowerApps-Completion-Guide.md for full 87KB screen-by-screen build) |
| `04-Documentation/` | SDD, test plan (76 cases), user guide, tear sheet, Appkit4 colour map |
| `05-Reference-Docs/` | Original project artifacts (HTML/DOCX) — read-only reference, no modifications needed |
| `06-Environment-Config/` | Environment variable template and find-and-replace checklist for tenant customization |

## Key Data Model

- **OFR_Issues** (11 columns): Title, ItemID (`OFR-NNN`), Owner, Priority (High/Medium/Low), Status (New/Active/Monitoring/Escalated/Closed), DateRaised, LastUpdated, NextAction, DaysSinceUpdate, StalenessFlag (Current/Aging/Stale), FunctionalGroup (10 groups)
- **OFR_UpdateHistory** (6 columns): Append-only audit trail linked by ParentItemID
- **OFR_IntakeQueue** (7 columns): Triage queue with TriageStatus (Pending/Promoted/Dismissed/Accepted/Rejected)

Relationship: `OFR_Issues.ItemID` ↔ `OFR_UpdateHistory.ParentItemID` (logical, not enforced by SharePoint).

## Staleness Logic

Calculated daily by Power Automate: `DaysSinceUpdate = days since LastUpdated`. Thresholds: 0-7 days = Current, 8-14 = Aging, 15+ = Stale.

## Environment Placeholders

Documentation uses `[TENANT]`, `[TENANT-DOMAIN]`, `[ADMIN-EMAIL]`, `[ADMIN-DISPLAY-NAME]` as placeholders. Bulk replacement commands are in `06-Environment-Config/find-and-replace-checklist.md`. The `[AUTO-GENERATED-*]` placeholders must NOT be pre-replaced — they are filled after component creation.

## Deployment Order (Dependencies)

Environment Variables → SharePoint Site + 3 Lists → (Optional Sample Data) → Both Power Automate Flows → Power Apps Canvas App → Share → Test → (Optional Teams Tab)

Power Apps depends on both the SharePoint lists (data sources) and the Intake Promotion flow (Promote button).

## Design System

Uses **Appkit4** colour theme (Blue/Orange/Red/Black). Full colour map with RGBA values and Power Apps Switch formulas is in `04-Documentation/OFR-Appkit4-Colour-Map.md`. Key semantic colours:
- Priority: High=Red `#E0301E`, Medium=Orange `#E45C2B`, Low=Blue `#415385`
- Staleness: Current=Blue, Aging=Orange, Stale=Red

## Working with This Repo

- All Power Apps formulas use `{Value: "text"}` syntax for Choice columns in Patch operations
- Flow expressions in `02-PowerAutomate/flow-expressions/*.txt` are copy-paste ready — do not modify
- The `05-Reference-Docs/` directory is read-only reference material from the original project
