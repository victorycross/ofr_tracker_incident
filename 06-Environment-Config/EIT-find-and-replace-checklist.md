# Enterprise Incident Tracker — Find-and-Replace Checklist

Use this checklist when deploying to a new M365 tenant. Replace all placeholder values with your actual environment values from `EIT-ENVIRONMENT-VARIABLES.md` before following any build guide.

---

## Placeholders Reference

| Placeholder | Replace With | Where Used |
|-------------|-------------|------------|
| `brightpathtechnology` | Your M365 tenant prefix | SharePoint URLs in all build guides |
| `brightpathtechnology.io` | Your primary email domain | Contact emails, shared mailbox |
| `david@brightpathtechnology.io` | Admin account email | Flow connection setup, test plan |
| `David` | Admin display name | Sample data owner fields |
| `[NS-LEAD-NAME]` | National Security lead name | EIT_DomainContacts row |
| `[NS-LEAD-EMAIL]` | National Security lead email | EIT_DomainContacts row, flow test |
| `[NIS-LEAD-NAME]` | NIS lead name | EIT_DomainContacts row |
| `[NIS-LEAD-EMAIL]` | NIS lead email | EIT_DomainContacts row, flow test |
| `[PT-LEAD-NAME]` | Privacy & Technology lead name | EIT_DomainContacts row |
| `[PT-LEAD-EMAIL]` | Privacy & Technology lead email | EIT_DomainContacts row, flow test |
| `[HC-LEAD-NAME]` | Human Capital lead name | EIT_DomainContacts row |
| `[HC-LEAD-EMAIL]` | Human Capital lead email | EIT_DomainContacts row, flow test |
| `[PS-LEAD-NAME]` | Physical Security lead name | EIT_DomainContacts row |
| `[PS-LEAD-EMAIL]` | Physical Security lead email | EIT_DomainContacts row, flow test |
| `[EXEC-ESCALATION-EMAIL]` | Executive escalation email | EIT_DomainContacts rows (EscalationEmail column) |
| `[EIT-PAC-GROUP-ID]` | Entra group GUID for P&C access group | Power Apps App.OnStart formula |
| `[AUTO-GENERATED-*]` | **Do not replace** — assigned by platform | Post-creation recording only |

---

## File-by-File Checklist

### 01-SharePoint/

| File | Placeholders | Status |
|------|-------------|--------|
| `EIT-CREATE-SITE.md` | `brightpathtechnology` | [ ] |
| `EIT_Incidents-schema.json` | *(none — schema only, no URLs)* | No changes needed |
| `EIT_IncidentHistory-schema.json` | *(none)* | No changes needed |
| `EIT_IntakeQueue-schema.json` | *(none)* | No changes needed |
| `EIT_DomainContacts-schema.json` | `[NS-LEAD-NAME]`, `[NS-LEAD-EMAIL]`, `[NIS-LEAD-NAME]`, `[NIS-LEAD-EMAIL]`, `[PT-LEAD-NAME]`, `[PT-LEAD-EMAIL]`, `[HC-LEAD-NAME]`, `[HC-LEAD-EMAIL]`, `[PS-LEAD-NAME]`, `[PS-LEAD-EMAIL]`, `[EXEC-ESCALATION-EMAIL]` | [ ] |
| `sample-data/EIT_DomainContacts-sample.csv` | All domain lead placeholders, `[EXEC-ESCALATION-EMAIL]` | [ ] |

> **Note on sample data CSVs:** `EIT_Incidents-sample.csv`, `EIT_IncidentHistory-sample.csv`, and `EIT_IntakeQueue-sample.csv` use `David` in owner/updatedBy fields. Replace before loading, or load as-is for testing and update owner values manually.

### 02-PowerAutomate/

| File | Placeholders | Status |
|------|-------------|--------|
| `EIT-Daily-Staleness-Escalation-Calculator.md` | `brightpathtechnology` | [ ] |
| `EIT-Intake-Promotion.md` | `brightpathtechnology` | [ ] |
| `EIT-Domain-Escalation-Notifier.md` | `brightpathtechnology`, `brightpathtechnology.io` (shared mailbox reference) | [ ] |
| `flow-expressions/EIT-promotion-ItemID.txt` | *(none — ready to use)* | No changes needed |
| `flow-expressions/EIT-escalation-Threshold.txt` | *(none — ready to use)* | No changes needed |

### 03-PowerApps/

| File | Placeholders | Status |
|------|-------------|--------|
| `EIT-REBUILD-GUIDE.md` | `brightpathtechnology`, `[EIT-PAC-GROUP-ID]` | [ ] |
| `EIT-PowerApps-Completion-Guide.md` | *(contains no environment-specific URLs — formulas reference data source names)* | No changes needed |

### 04-Documentation/

| File | Placeholders | Status |
|------|-------------|--------|
| `EIT-SDD.md` | `brightpathtechnology`, `brightpathtechnology.io`, `david@brightpathtechnology.io` | [ ] |
| `EIT-Test-Plan.md` | `brightpathtechnology`, `brightpathtechnology.io`, `david@brightpathtechnology.io` | [ ] |
| `EIT-User-Guide.md` | `brightpathtechnology` | [ ] |
| `EIT-Access-Control-Guide.md` | `brightpathtechnology`, `[EIT-PAC-GROUP-ID]` | [ ] |
| `EIT-Enterprise-Taxonomy.md` | *(none — reference doc only)* | No changes needed |
| `EIT-Escalation-Thresholds.md` | *(none)* | No changes needed |
| `EIT-Leadership-Report-Guide.md` | *(none)* | No changes needed |
| `EIT-Phase2-Integration-Roadmap.md` | `brightpathtechnology.io` (if ServiceNow URL referenced) | [ ] Check |

### 06-Environment-Config/

| File | Action | Status |
|------|--------|--------|
| `EIT-ENVIRONMENT-VARIABLES.md` | Fill in the "Your Value" column | [ ] |

---

## Quick Bulk Find-and-Replace (Terminal)

Run from the root of the project directory. Substitute your actual values on the right side of each command.

```bash
# Navigate to project directory
cd /path/to/OFR_tracker_incident

# Replace brightpathtechnology in all EIT .md files
find . -name "EIT*" -exec sed -i '' 's|\[TENANT\]|contoso|g' {} +

# Replace brightpathtechnology.io
find . -name "EIT*" -exec sed -i '' 's|\[TENANT-DOMAIN\]|contoso.com|g' {} +

# Replace david@brightpathtechnology.io
find . -name "EIT*" -exec sed -i '' 's|\[ADMIN-EMAIL\]|admin@contoso.com|g' {} +

# Replace David
find . -name "EIT*" -exec sed -i '' 's|\[ADMIN-DISPLAY-NAME\]|Jane Smith|g' {} +

# Replace [EXEC-ESCALATION-EMAIL]
find . -name "EIT*" -exec sed -i '' 's|\[EXEC-ESCALATION-EMAIL\]|exec-incidents@contoso.com|g' {} +

# Replace [EIT-PAC-GROUP-ID] (after creating the Entra group)
find . -name "EIT*" -exec sed -i '' 's|\[EIT-PAC-GROUP-ID\]|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|g' {} +

# Replace domain lead names and emails (repeat pattern for each domain)
find . -name "EIT*" -exec sed -i '' 's|\[NS-LEAD-NAME\]|Alex Chen|g' {} +
find . -name "EIT*" -exec sed -i '' 's|\[NS-LEAD-EMAIL\]|alex.chen@contoso.com|g' {} +
# ... repeat for NIS, PT, HC, PS leads
```

> **Warning:** Do NOT replace `[AUTO-GENERATED-*]` placeholders. These are assigned by the platform when you create components and cannot be pre-set.

---

## Verification

After completing all replacements, run this command to confirm no EIT placeholders remain:

```bash
grep -r '\[TENANT\]\|\[TENANT-DOMAIN\]\|\[ADMIN-EMAIL\]\|\[ADMIN-DISPLAY-NAME\]\|\[EXEC-ESCALATION-EMAIL\]\|\[EIT-PAC-GROUP-ID\]' . \
  --include="EIT*.md" \
  --include="EIT*.json" \
  --include="EIT*.csv"
```

This should return zero results (excluding `EIT-ENVIRONMENT-VARIABLES.md` and this checklist file itself).

---

## Post-Deployment Recording

After building each component, record the auto-generated IDs in `EIT-ENVIRONMENT-VARIABLES.md`:

- [ ] Power Apps App ID — from Power Apps Studio → App details
- [ ] List GUIDs — from SharePoint list URL after `/Lists/`
- [ ] Flow IDs — from Power Automate flow detail URL
- [ ] EIT-PAC-GROUP-ID — from Entra admin portal → Groups → Group object ID
