# Enterprise Incident Tracker — Power BI Row-Level Security Guide

**Version:** 1.0
**Date:** February 2026
**Audience:** Power BI workspace administrator, EIT administrator
**Classification:** Internal

---

## Overview

The EIT Power BI dashboard connects to the `EIT_Incidents` SharePoint list, which contains incidents marked Privileged & Confidential (`PrivilegedAndConfidential = Yes`). These incidents must not be visible to broad audiences on the shared dashboard.

Row-Level Security (RLS) is the Power BI mechanism to enforce this restriction. This guide covers:

1. What RLS does and how it applies to the EIT dashboard
2. Setting up the Standard User role in Power BI Desktop
3. Publishing RLS to Power BI Service
4. Assigning users to roles in Power BI Service
5. Testing RLS enforcement
6. The alternative: separate P&C dataset

---

## How EIT RLS Works

| Audience | RLS Role | P&C Incidents Visible? |
|----------|----------|----------------------|
| Standard users (EIT Members, EIT Viewers) | `Standard_Users` | No — filtered out by RLS |
| P&C authorized users (EIT-PAC-Authorized group) | No role assigned *(see all data)* | Yes |
| Power BI workspace admins | No role assigned | Yes |

**Key principle:** Users with no RLS role assignment in the Power BI workspace see all data. Therefore:
- Standard users must be **assigned to the `Standard_Users` role**
- P&C authorized users must **not be assigned to any role** (they see everything)
- Workspace admins automatically bypass RLS

---

## Step 1: Create the RLS Role in Power BI Desktop

RLS roles are created in the PBIX file before publishing.

1. Open the EIT Power BI report (`EIT-Dashboard.pbix`) in **Power BI Desktop**
2. Click the **Modeling** tab in the ribbon
3. Click **Manage roles**
4. Click **Create**
5. Role name: `Standard_Users`
6. In the table list, find **EIT_Incidents**
7. In the **Table filter DAX expression** field, enter:

```
[PrivilegedAndConfidential] = FALSE
```

> **Column name:** The `PrivilegedAndConfidential` column is a Yes/No (boolean) column in SharePoint. Power BI imports it as a boolean. Use `= FALSE` (not `= "No"` or `= 0`). If the column was imported as text, use `[PrivilegedAndConfidential] <> "Yes"` instead.

8. Click the **checkmark** to validate the expression
9. Click **Save**

---

## Step 2: Test RLS in Power BI Desktop

Before publishing, test the role to confirm P&C incidents are excluded.

1. In Power BI Desktop, click **Modeling** → **View as roles**
2. Check **Standard_Users**
3. Click **OK**

The report now renders as a Standard User would see it. Verify:
- P&C incidents do not appear in any visuals or tables
- Non-P&C incident counts and charts are correct
- No error messages are displayed

4. Click **Stop viewing** to return to admin view

---

## Step 3: Publish to Power BI Service

1. Click **File** → **Publish** → **Publish to Power BI**
2. Select the **Enterprise Incident Tracker** workspace (create it if it does not exist)
3. Click **Select** and wait for publishing to complete

The RLS role configuration is included in the published PBIX.

---

## Step 4: Assign Users to the Standard_Users Role in Power BI Service

1. Navigate to `https://app.powerbi.com`
2. Open the **Enterprise Incident Tracker** workspace
3. Find the **EIT-Dashboard** dataset (not the report — the dataset)
4. Click the three-dot menu → **Security**
5. In the Row-Level Security panel:
   - Select the **Standard_Users** role
   - In the **Members** field, add:
     - The `EIT Members` M365 group (e.g., `EnterpriseIncidentTrackerMembers@brightpathtechnology.io`)
     - The `EIT Viewers` M365 group (e.g., `EnterpriseIncidentTrackerViewers@brightpathtechnology.io`)
6. Click **Save**

**Do not add** the `EIT-PAC-Authorized` group or EIT Owners to the `Standard_Users` role — they need unrestricted access.

> **M365 group vs individual users:** Adding M365 security groups is preferred over individual user assignments. Group membership is managed centrally in Entra ID; adding or removing a user from the M365 group automatically updates their Power BI RLS access without modifying the Power BI security configuration.

---

## Step 5: Assign P&C-Authorized Users

P&C authorized users (the `EIT-PAC-Authorized` M365 group) must be added to the Power BI workspace as **members** but must **not** be assigned to any RLS role. This gives them full, unfiltered access to the dataset.

1. In the **Enterprise Incident Tracker** workspace, click **Access**
2. Add the `EIT-PAC-Authorized` group with **Member** permission
3. Confirm they are **not** listed in the **Standard_Users** role under Security

---

## Step 6: Test RLS in Power BI Service

Test with a Standard User account to confirm enforcement:

1. Open the EIT dashboard report
2. Click the three-dot menu → **Test as role**
3. Select **Standard_Users**
4. Click **Apply**

Verify:
- P&C incidents are not visible in any table, chart, or filter
- Overall counts on the Executive Overview page are lower than the admin view (the difference = P&C incident count)
- Escalation Drill-Down does not show P&C incidents

Then test with a P&C-authorized account:
- Sign in to `https://app.powerbi.com` as a user in the `EIT-PAC-Authorized` group
- Open the EIT dashboard
- Verify P&C incidents are visible

---

## Alternative Pattern: Separate P&C Dataset

For environments requiring stronger separation, consider maintaining two separate Power BI datasets:

| Dataset | Audience | P&C incidents |
|---------|----------|--------------|
| `EIT-Dashboard` | All EIT users | Excluded via RLS |
| `EIT-Dashboard-PAC` | P&C authorized only | Included (no RLS) |

The `EIT-Dashboard-PAC` report connects to the same SharePoint data but the workspace is restricted to P&C authorized users only (Private workspace, no broader sharing). This provides defence in depth beyond RLS.

---

## Maintaining RLS After Changes

| Event | Action Required |
|-------|----------------|
| New member added to EIT Members group | Automatic — M365 group membership updates RLS immediately |
| New P&C authorized user | Add to `EIT-PAC-Authorized` M365 group. Do NOT add to `Standard_Users` role. |
| User leaves EIT | Remove from relevant M365 groups. Power BI access is revoked automatically. |
| New incident classified as P&C | No Power BI action needed — SharePoint `PrivilegedAndConfidential` flag controls filtering at dataset refresh |
| Report re-published (PBIX updated) | Re-check RLS role assignments in Power BI Service after each publish. Publishing does not overwrite role assignments, but verify after major model changes. |

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Standard users see P&C incidents | Verify they are assigned to `Standard_Users` role in Power BI Service dataset security. If added as a workspace Admin or Member with no role, they bypass RLS. |
| P&C users see filtered data | Confirm they are NOT assigned to `Standard_Users` role. Check dataset security panel. |
| `[PrivilegedAndConfidential] = FALSE` returns an error | The column may have been imported as text. Change the DAX filter to `[PrivilegedAndConfidential] <> "Yes"`. |
| RLS role not appearing after publish | Ensure the role was saved in Power BI Desktop before publishing. Re-open the PBIX and check Modeling → Manage roles. |
| M365 group not accepted in role assignment | Groups must be mail-enabled or security groups in Entra ID. Distribution lists are not supported. Use `EIT Members` and `EIT Viewers` security groups. |
