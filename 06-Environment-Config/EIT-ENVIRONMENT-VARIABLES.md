# Enterprise Incident Tracker — Environment Variables

Fill in this table with your target environment values before deploying. Replace every `[PLACEHOLDER]` in the EIT documentation with the values below.

---

## Required Substitutions

| Placeholder | Your Value | Description |
|-------------|-----------|-------------|
| `brightpathtechnology` | __________ | M365 tenant prefix (e.g., `contoso`). Subdomain in `contoso.sharepoint.com`. |
| `brightpathtechnology.io` | __________ | Primary email domain (e.g., `contoso.com`). Used for contact email addresses and shared mailbox. |
| `david@brightpathtechnology.io` | __________ | Admin account email for Power Platform connections (e.g., `admin@contoso.com`). This account must have SharePoint Site Owner and Exchange Online mailbox access. |
| `David` | __________ | Display name of the admin account. Used in sample data owner fields. |
| `[NS-LEAD-NAME]` | __________ | Full name of the National Security domain lead. |
| `[NS-LEAD-EMAIL]` | __________ | Email address of the National Security domain lead. |
| `[NIS-LEAD-NAME]` | __________ | Full name of the NIS domain lead. |
| `[NIS-LEAD-EMAIL]` | __________ | Email address of the NIS domain lead. |
| `[PT-LEAD-NAME]` | __________ | Full name of the Privacy & Technology domain lead. |
| `[PT-LEAD-EMAIL]` | __________ | Email address of the Privacy & Technology domain lead. |
| `[HC-LEAD-NAME]` | __________ | Full name of the Human Capital domain lead. |
| `[HC-LEAD-EMAIL]` | __________ | Email address of the Human Capital domain lead. |
| `[PS-LEAD-NAME]` | __________ | Full name of the Physical Security domain lead. |
| `[PS-LEAD-EMAIL]` | __________ | Email address of the Physical Security domain lead. |
| `[EXEC-ESCALATION-EMAIL]` | __________ | Email address for Executive Brief escalations (e.g., a shared exec mailbox like `exec-incidents@contoso.com`). All 5 domains use the same value unless overridden. |
| `[EIT-PAC-GROUP-ID]` | __________ | Entra ID object GUID for the P&C-authorized access group. Obtain from Azure portal → Groups after creating the group. Used in Power Apps `App.OnStart` formula. |

---

## Derived URLs (Computed from Above)

| Resource | URL |
|----------|-----|
| SharePoint Site | `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker` |
| EIT_Incidents List | `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker/Lists/EIT_Incidents` |
| EIT_IncidentHistory List | `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker/Lists/EIT_IncidentHistory` |
| EIT_IntakeQueue List | `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker/Lists/EIT_IntakeQueue` |
| EIT_DomainContacts List | `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker/Lists/EIT_DomainContacts` |
| Power Apps Studio | `https://make.powerapps.com` |
| Power Automate | `https://make.powerautomate.com` |
| Entra ID Groups | `https://entra.microsoft.com/#view/Microsoft_AAD_IAM/GroupsManagementMenuBlade` |
| Microsoft Purview (Audit) | `https://compliance.microsoft.com` |

---

## Auto-Generated IDs (Do Not Pre-Set)

These identifiers are automatically assigned when components are created. Record them here after creation.

| Component | ID |
|-----------|----|
| Power Apps App ID | *(recorded after app creation)* |
| EIT_Incidents List GUID | *(recorded after list creation)* |
| EIT_IncidentHistory List GUID | *(recorded after list creation)* |
| EIT_IntakeQueue List GUID | *(recorded after list creation)* |
| EIT_DomainContacts List GUID | *(recorded after list creation)* |
| Staleness & Escalation Calculator Flow ID | *(recorded after flow creation)* |
| Intake Promotion Flow ID | *(recorded after flow creation)* |
| Domain Escalation Notifier Flow ID | *(recorded after flow creation)* |
| EIT-PAC-GROUP-ID (Entra group GUID) | *(recorded after group creation — required for App.OnStart formula)* |

---

## M365 Permission Groups

Three SharePoint permission groups must be created in Entra ID before site provisioning. See `04-Documentation/EIT-Access-Control-Guide.md` for full setup instructions.

| Group Name | SharePoint Role | Population |
|-----------|-----------------|------------|
| `EIT Members` | Contribute | All domain leads, incident owners, operational staff |
| `EIT Viewers` | Read | Executive stakeholders (read-only) |
| `EIT Owners` | Full Control | EIT administrators (1–2 people) |

Additionally, create one more group:

| Group Name | Purpose |
|-----------|---------|
| `EIT-PAC-Authorized` | Members can view P&C incidents in the Power Apps UI. Record the GUID as `[EIT-PAC-GROUP-ID]`. |

---

## Licensing Requirements

| Component | License Required | Notes |
|-----------|-----------------|-------|
| SharePoint Online | M365 Business Standard (included) | 4 lists, item-level permissions |
| Power Apps | Seeded M365 plan or Power Apps per-user | Canvas app, 4 data sources |
| Power Automate Standard | Seeded M365 Business Standard | Required for Outlook Send an email (V2) connector |
| Entra ID | Included with M365 | Group membership check via Office365Groups connector |
| Microsoft Purview | E3/E5 for 1-year audit log retention; Business Standard = 90 days | Audit log review for P&C access events |
| Office 365 Groups connector | Included with M365 | Used in Power Apps `App.OnStart` for P&C group check |

---

## Deployment Order Reminder

1. Fill in all values in this file
2. Create M365 groups (EIT Members, EIT Viewers, EIT Owners, EIT-PAC-Authorized)
3. Run find-and-replace using `EIT-find-and-replace-checklist.md`
4. Create SharePoint site → 4 lists → load sample data
5. Create EIT_DomainContacts rows with real lead names and emails
6. Build 3 Power Automate flows
7. Build Power Apps canvas app, connect data sources and flows
8. Set App.OnStart with `[EIT-PAC-GROUP-ID]` value
9. Share app with EIT Members group
10. Test end-to-end, then publish
