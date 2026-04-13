# OFR Issue Tracker — Environment Variables

Fill in this table with your target environment values before deploying.

## Required Substitutions

| Placeholder | Your Value | Description |
|-------------|-----------|-------------|
| `[TENANT]` | __________ | M365 tenant prefix (e.g., `contoso`). This is the subdomain in `contoso.sharepoint.com`. |
| `[TENANT-DOMAIN]` | __________ | Primary email domain (e.g., `contoso.com`). Used for admin email addresses. |
| `[ADMIN-EMAIL]` | __________ | Admin account email for Power Platform connections (e.g., `admin@contoso.com`). |
| `[ADMIN-DISPLAY-NAME]` | __________ | Display name of the admin account (e.g., `Jane Smith`). Used in sample data. |
| `[M365-GROUP]` | `OFR Issue Tracker Members` | M365 security group for site access. Auto-created with the SharePoint team site. Change if using a different group name. |

## Derived URLs (Computed from Above)

| URL | Value |
|-----|-------|
| SharePoint Site | `https://[TENANT].sharepoint.com/sites/OFRIssueTracker` |
| OFR_Issues List | `https://[TENANT].sharepoint.com/sites/OFRIssueTracker/Lists/OFR_Issues` |
| OFR_UpdateHistory List | `https://[TENANT].sharepoint.com/sites/OFRIssueTracker/Lists/OFR_UpdateHistory` |
| OFR_IntakeQueue List | `https://[TENANT].sharepoint.com/sites/OFRIssueTracker/Lists/OFR_IntakeQueue` |
| Power Apps Studio | `https://make.powerapps.com` |
| Power Automate | `https://make.powerautomate.com` |

## Auto-Generated IDs (Do Not Pre-Set)

The following identifiers are automatically generated when components are created. You do **not** need to set these — they will be assigned by the platform. Record them here after creation for your reference.

| Component | New ID |
|-----------|--------|
| Power Apps App ID | *(recorded after app creation)* |
| OFR_Issues List GUID | *(recorded after list creation)* |
| OFR_UpdateHistory List GUID | *(recorded after list creation)* |
| OFR_IntakeQueue List GUID | *(recorded after list creation)* |
| Staleness Calculator Flow ID | *(recorded after flow creation)* |
| Intake Promotion Flow ID | *(recorded after flow creation)* |

## Original Environment Reference

For reference, the original deployment used these values:

| Placeholder | Original Value |
|-------------|---------------|
| `[TENANT]` | `papercutscafe` |
| `[TENANT-DOMAIN]` | `papercuts.cafe` |
| `[ADMIN-EMAIL]` | `david@papercuts.cafe` |
| `[ADMIN-DISPLAY-NAME]` | `David` |
| Power Apps App ID | `0fbbc26c-ad71-476a-bcfc-edc0d7989533` |
| OFR_Issues List GUID | `a70da6a6-1f0a-4fd3-bb4e-cf7847e18a99` |
| Staleness Flow ID | `aefb8de0-35fe-4d5d-a629-ddd8502ee5aa` |
| Intake Promotion Flow ID | `1c631640-113f-4602-805e-1d693582de8c` |

## Licensing Requirements

| Component | License Required |
|-----------|-----------------|
| SharePoint Online | M365 Business Standard (included) |
| Power Apps | Power Apps Developer Plan (free) or Power Apps per-user |
| Power Automate | Power Automate Free (included with M365) |
