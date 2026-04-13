# OFR Issue Tracker — Find-and-Replace Checklist

Use this checklist when deploying to a new environment. Replace all placeholder values with your actual environment values from `ENVIRONMENT-VARIABLES.md`.

---

## Placeholders

| Placeholder | Replace With | Example |
|-------------|-------------|---------|
| `[TENANT]` | Your M365 tenant prefix | `contoso` |
| `[TENANT-DOMAIN]` | Your primary domain | `contoso.com` |
| `[ADMIN-EMAIL]` | Your admin account email | `admin@contoso.com` |
| `[ADMIN-DISPLAY-NAME]` | Your admin display name | `Jane Smith` |
| `[M365-GROUP]` | Your M365 group name | `OFR Issue Tracker Members` |
| `[AUTO-GENERATED-APP-ID]` | *(Do not replace — new ID assigned on creation)* | — |
| `[AUTO-GENERATED-LIST-GUID]` | *(Do not replace — new ID assigned on creation)* | — |
| `[AUTO-GENERATED-FLOW-ID]` | *(Do not replace — new ID assigned on creation)* | — |

---

## File-by-File Checklist

### 01-SharePoint/

| File | Placeholders to Replace | Status |
|------|------------------------|--------|
| `CREATE-SITE.md` | `[TENANT]` | [ ] |

### 02-PowerAutomate/

| File | Placeholders to Replace | Status |
|------|------------------------|--------|
| `OFR-Daily-Staleness-Calculator.md` | `[TENANT]` | [ ] |
| `OFR-Intake-Promotion.md` | `[TENANT]` | [ ] |

> **Note:** The flow expression `.txt` files do NOT contain any environment-specific values. They are ready to use as-is.

### 03-PowerApps/

| File | Placeholders to Replace | Status |
|------|------------------------|--------|
| `REBUILD-GUIDE.md` | `[TENANT]` | [ ] |
| `OFR-PowerApps-Completion-Guide.md` | *(Contains no environment-specific URLs — formulas reference data source names, not URLs)* | No changes needed |

### 04-Documentation/

| File | Placeholders to Replace | Status |
|------|------------------------|--------|
| `OFR-SDD.md` | `[TENANT]`, `[TENANT-DOMAIN]`, `[ADMIN-EMAIL]`, `[AUTO-GENERATED-*]` | [ ] Already neutralized |
| `OFR-Completion-Guide.md` | `[TENANT]`, `[TENANT-DOMAIN]`, `[ADMIN-EMAIL]`, `[AUTO-GENERATED-*]` | [ ] Already neutralized |
| `OFR-Test-Plan.md` | *(Verify — may contain no environment refs)* | [ ] Check |
| `OFR-User-Guide.md` | *(Verify — may contain no environment refs)* | [ ] Check |
| `OFR-Tear-Sheet.html` | *(Verify — may contain no environment refs)* | [ ] Check |

### 05-Reference-Docs/

| File | Placeholders to Replace | Status |
|------|------------------------|--------|
| All files | *(Reference only — no substitution needed. These are original artifacts.)* | No changes needed |

### 06-Environment-Config/

| File | Placeholders to Replace | Status |
|------|------------------------|--------|
| `ENVIRONMENT-VARIABLES.md` | Fill in the "Your Value" column | [ ] |

---

## Quick Find-and-Replace Commands

If you prefer to do bulk replacements from a terminal, you can use these commands (replace the values on the right side with your actual values):

```bash
# Navigate to the package directory
cd OFR-Migration-Package

# Replace [TENANT] in all .md files
find . -name "*.md" -exec sed -i '' 's|\[TENANT\]|contoso|g' {} +

# Replace [TENANT-DOMAIN] in all .md files
find . -name "*.md" -exec sed -i '' 's|\[TENANT-DOMAIN\]|contoso.com|g' {} +

# Replace [ADMIN-EMAIL] in all .md files
find . -name "*.md" -exec sed -i '' 's|\[ADMIN-EMAIL\]|admin@contoso.com|g' {} +

# Replace [ADMIN-DISPLAY-NAME] in all .md files
find . -name "*.md" -exec sed -i '' 's|\[ADMIN-DISPLAY-NAME\]|Jane Smith|g' {} +
```

> **Warning:** Do NOT replace `[AUTO-GENERATED-*]` placeholders. These values are assigned by the platform when you create the components and cannot be pre-set.

---

## Verification After Replacement

After completing all replacements:

```bash
# Check for any remaining placeholders
grep -r '\[TENANT\]\|\[TENANT-DOMAIN\]\|\[ADMIN-EMAIL\]\|\[ADMIN-DISPLAY-NAME\]' . --include="*.md"
```

This command should return zero results (excluding this checklist file itself and the ENVIRONMENT-VARIABLES.md file).
