# Enterprise Incident Tracker — Access Control & Privilege Guide

**Version:** 1.0
**Date:** February 2026
**Classification:** Internal

---

## Overview

The EIT uses a two-layer access control model:

1. **List-level access** — Controlled by SharePoint permission groups. Determines who can see the EIT site and lists at all.
2. **Item-level access** — Applied manually to individual incident records. Used for incidents marked Privileged & Confidential (P&C) or for incidents involving specific named individuals.

These two layers work together. A user must pass both layers to view a given incident record.

---

## Layer 1 — SharePoint Permission Groups

Three M365 groups control baseline access to the EIT SharePoint site:

| Group Name | SharePoint Role | Who Belongs Here |
|---|---|---|
| `EIT Members` | Contribute | All domain leads, incident owners, and operational staff who submit or update incidents |
| `EIT Viewers` | Read | Executive stakeholders and leadership who need read-only visibility |
| `EIT Owners` | Full Control | EIT administrators only — typically 1-2 people responsible for maintaining the site, lists, and permissions |

**What each role can do:**

| Action | EIT Members | EIT Viewers | EIT Owners |
|---|---|---|---|
| Submit new intake items | Yes | No | Yes |
| View all non-P&C incidents | Yes | Yes | Yes |
| Update incident records | Yes | No | Yes |
| Triage intake queue | Domain leads only (by convention) | No | Yes |
| Set P&C flag | Domain leads only (by convention) | No | Yes |
| Manage item-level permissions | No | No | Yes |
| Modify lists or columns | No | No | Yes |

### Setting Up Permission Groups

1. Navigate to `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
2. Click the **gear icon** → **Site permissions**
3. Click **Advanced permissions settings**
4. Use **Grant Permissions** to add the appropriate M365 groups:
   - Add `EIT Members` M365 group → **Contribute**
   - Add `EIT Viewers` M365 group → **Read**
5. Remove the default `EnterpriseIncidentTracker Members` group's Contribute access if it includes too broad a population
6. Verify that `EIT Owners` maps to the SharePoint **Owners** group (default)

> **Important:** The Power Apps canvas app uses the SharePoint connection of the signed-in user. If a user does not have at least Contribute access to the SharePoint site, the app will fail to load data. All app users must be in `EIT Members` or `EIT Owners`.

---

## Layer 2 — Item-Level Permissions for P&C Incidents

Item-level permissions in SharePoint allow a specific list row to have different permissions than the rest of the list. This is the mechanism for restricting access to P&C incidents to a need-to-know group.

### When to Apply Item-Level Permissions

Apply item-level permissions to an `EIT_Incidents` row when:
- `PrivilegedAndConfidential` is set to **Yes**, AND
- The incident involves legal counsel communications, regulatory proceedings, employee litigation, or executive investigations

Do **not** apply item-level permissions solely because `ConfidentialityLevel = Confidential`. Confidentiality level is an informational classification; it does not restrict access at the SharePoint layer. Only P&C designation triggers the item-level restriction process.

### Step-by-Step: Restricting a P&C Incident

Perform these steps immediately after setting `PrivilegedAndConfidential = Yes` on an incident:

1. Navigate to the `EIT_Incidents` list in SharePoint
2. Locate the incident row (filter by ItemID or use list view)
3. Click the **...** (three dots) on the row → **More** → **Manage access**
   - Alternatively: open the item → click the **information icon (ⓘ)** → **Manage access**
4. Click **Stop inheriting permissions**
   - Confirm the warning dialog — this creates a unique permission copy for this item
5. Remove groups that should not have access:
   - Select `EIT Members` → **Remove**
   - Select `EIT Viewers` → **Remove**
   - Keep `EIT Owners` (administrators must retain access)
6. Add the authorized individuals and groups:
   - Click **Grant access**
   - Add each authorized person by name or email
   - Set their permission level: **Contribute** (if they need to update) or **Read** (if view-only)
   - Click **Share**
7. Document the access grant: add a note in `EIT_IncidentHistory` recording who was granted access, by whom, and why — with `PrivilegedAndConfidential = Yes` on that history entry

### P&C Authorized Access — Who to Include

The minimum access list for any P&C incident should include:

| Role | Access Level |
|---|---|
| Incident Owner | Contribute |
| Domain Lead for the incident's domain | Contribute |
| Legal counsel of record (OGC or external) | Contribute or Read |
| EIT Administrator | Full Control (via EIT Owners group — already present) |
| Executive sponsor (if escalated to Executive Brief) | Read |

Add additional individuals only with explicit approval from the domain lead or legal counsel. Document every addition in `EIT_IncidentHistory`.

### Restoring Inherited Permissions (When P&C is Cleared)

If a P&C designation is removed (i.e., `PrivilegedAndConfidential` is set back to **No**):

1. Navigate to the incident row in the `EIT_Incidents` list
2. Open **Manage access** for the item
3. Click **Delete unique permissions** → confirm
   - The item now inherits list-level permissions again — all `EIT Members` can see it
4. Add a history entry in `EIT_IncidentHistory` recording the P&C clearance, who authorized it, and the date

> **Caution:** Clearing P&C and restoring inherited permissions makes the incident visible to all `EIT Members`. Confirm with the domain lead and legal counsel before doing so.

---

## The Privileged & Confidential Designation

### What It Means

"Privileged and Confidential" is a legal designation indicating that a communication or document is protected by **attorney-client privilege** and/or the **work product doctrine**. In the EIT context:

- **Attorney-client privilege** applies to communications made for the purpose of obtaining or giving legal advice between a client (the firm) and its legal counsel (OGC or external counsel).
- **Work product doctrine** applies to materials prepared in anticipation of litigation or regulatory proceedings.

These protections can be waived if privileged materials are disclosed to unauthorized parties. Misuse of the P&C designation — applying it to non-privileged content to shield it from disclosure — can expose the firm to adverse legal inferences.

### When to Use the P&C Flag

**Use it when the incident:**
- Has been referred to OGC (Office of General Counsel) or external legal counsel for legal advice
- Involves ongoing or anticipated litigation, arbitration, or regulatory enforcement proceedings
- Contains communications between the incident owner and legal counsel (in-house or external)
- Is subject to a legal hold order
- Involves regulatory proceedings where legal advice is being sought (e.g., PIPEDA breach notification assessment with OGC, employment tribunal referral)

**Do NOT use it for:**
- Incidents that are merely sensitive, embarrassing, or reputationally risky
- Incidents classified as Confidential that do not involve legal proceedings
- Incidents involving executives — seniority alone does not confer privilege
- Incidents flagged as P&C pre-emptively "just in case" — privilege must be earned by the nature of the communication

### Who Can Set or Clear the P&C Flag

The P&C flag may only be set or cleared by:
- A **domain lead** for the incident's domain
- Legal counsel of record (OGC or external)
- An **EIT administrator**

Standard `EIT Members` users may suggest P&C designation at intake (via the `PrivilegedAndConfidential` field in `EIT_IntakeQueue`) but the triage reviewer must confirm or clear it during promotion.

### How the Flag Appears in the System

| Surface | P&C = No | P&C = Yes |
|---|---|---|
| Tracker screen | Normal row | Red `P&C` badge on the row |
| Issue Detail screen | Standard header | Amber warning banner at top of screen: "This incident is marked Privileged & Confidential. Access is restricted. Do not discuss or share outside the authorized list." |
| Intake Review panel | No indicator | P&C badge visible to triage reviewer |
| Escalation email subject | Normal | `[PRIVILEGED & CONFIDENTIAL]` prepended to subject line |
| Escalation email footer | Standard sign-off | Additional disclaimer: "Do not forward this email or discuss its contents with personnel not on the authorized access list." |
| History entries | Normal | P&C entries visually distinguished (grey italic) |

> **Note:** Power Apps cannot enforce item-level SharePoint permissions — it will still show P&C incidents in galleries if the user has list-level Contribute access. The item-level SharePoint permission restriction (Layer 2) is the enforceable control. Power Apps UI treatment (banners, badges) is a user experience layer only, not a security barrier.

---

## Domain-Based Access Restrictions

Beyond P&C, you may want to restrict incident visibility by domain — for example, ensuring that NIS staff can only see NIS incidents, not National Security incidents.

SharePoint's native permission model does not support row-level filtering by column value. Three practical approaches are available:

### Option A — Separate Lists per Domain (Maximum Isolation)

Create one `EIT_Incidents` list per domain (e.g., `EIT_Incidents_NS`, `EIT_Incidents_NIS`, etc.) with separate permission groups per list. The Power Apps app connects to all lists but each user only has access to their domain's list.

| Pros | Cons |
|---|---|
| True SharePoint-enforced isolation | 5× list management overhead |
| Domain leads cannot see other domains' data even in SharePoint | Cross-domain reporting requires connecting all 5 lists in Power Apps |
| | Phase 2 integration complexity increases |

### Option B — Single List with Power Apps Filtering (Default Recommendation)

Use a single `EIT_Incidents` list with list-level permissions for all `EIT Members`. In Power Apps, filter galleries by domain based on the signed-in user's domain group membership:

```
// In Power Apps: check if user is in a domain-specific group
// Example for TrackerScreen gallery Items formula:
Filter(
    EIT_Incidents,
    varDomainFilter = "All" || Domain.Value = varDomainFilter,
    // Only show P&C incidents to authorized users:
    Not(PrivilegedAndConfidential) || varUserHasPACAccess
)
```

Where `varUserHasPACAccess` is a context variable set on app start using:
```
Set(varUserHasPACAccess,
    Office365Groups.IsMemberOfGroup("[EIT-PAC-GROUP-ID]", User().Email).value
)
```

| Pros | Cons |
|---|---|
| Single list — simple maintenance | Power Apps filtering is a UX control, not a security barrier |
| Cross-domain reporting works natively | Determined users can bypass via direct SharePoint list access |
| Easier Phase 2 integration | Requires Entra group ID for group membership check |

**Recommendation:** Use Option B as the default for domain-level access. Use Option A only if legal or regulatory requirements mandate true data segregation between domains.

### Option C — SharePoint Column Permissions (Per-Column Restriction)

SharePoint allows hiding specific columns from users with certain permission levels. This can be used to hide `ConfidentialityLevel` or `NextAction` from `EIT Viewers` while still allowing them to see the incident exists.

Configure via **List Settings** → **Column settings** → set the column to hidden for specific views or use audience-targeted views. This is a view-level control, not a security control.

---

## Incidents Involving Specific Named Individuals

Incidents involving named individuals — particularly personnel investigations, harassment complaints, or executive conduct matters — require heightened access controls regardless of whether they are formally marked P&C.

### Principles

1. **Minimum necessary access:** Only individuals directly involved in managing the incident should have access. Other domain leads and standard `EIT Members` should not.
2. **Named individual exclusion:** The subject of a personnel investigation must not have access to their own incident record in EIT. Verify they are not in the authorized access list before setting item-level permissions.
3. **HR partnership:** Human Capital domain incidents involving named individuals should be managed jointly with the CHRO or designated HR lead. The EIT administrator should not unilaterally grant access to HC personnel matters.
4. **Document the access list:** Every personnel incident with restricted access must have an `EIT_IncidentHistory` entry listing the authorized access list by name at the time of restriction.

### Handling Personnel Incidents Step-by-Step

1. When a personnel incident is submitted, the triage reviewer assesses whether it involves a named individual in an investigation context
2. If yes: set `ConfidentialityLevel = Confidential` and consult the domain lead before deciding on P&C designation
3. If legal counsel has been engaged or litigation is anticipated: set `PrivilegedAndConfidential = Yes`
4. Apply item-level permissions immediately (see Layer 2 instructions above)
5. Verify the subject individual is not in the authorized access list
6. Coordinate with HR lead to confirm the approved access list
7. Document the access decision in `EIT_IncidentHistory` with `PrivilegedAndConfidential = Yes` on that history entry

---

## Auditing Access to P&C Incidents

SharePoint records all item-level access events in the **Unified Audit Log** (Microsoft Purview Compliance portal). To review access to a specific P&C incident:

1. Navigate to `https://compliance.microsoft.com`
2. Go to **Audit** → **Audit search**
3. Filter:
   - **Activities:** SharePoint file accessed, SharePoint file viewed
   - **Date range:** Incident date to present
   - **Site:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
4. Search and export results
5. Review for any access by users not on the authorized list — escalate anomalies to the EIT administrator and domain lead

> **Note:** Audit log retention defaults to 90 days for M365 Business Standard and 1 year for E3/E5. If regulatory requirements mandate longer retention, configure audit log retention policies in Purview before incidents are created.

---

## Summary Checklist — P&C Incident

When an incident is designated Privileged & Confidential, complete all of the following:

- [ ] Set `PrivilegedAndConfidential = Yes` on the `EIT_Incidents` row
- [ ] Set `ConfidentialityLevel = Confidential` (if not already)
- [ ] Apply item-level SharePoint permissions — stop inheritance, remove `EIT Members` and `EIT Viewers`, add authorized individuals only
- [ ] Verify the named subject (if any) is NOT on the access list
- [ ] Add a history entry in `EIT_IncidentHistory` documenting who is on the authorized access list, who granted access, and the date — set `PrivilegedAndConfidential = Yes` on this history entry
- [ ] Confirm legal counsel is on the access list with appropriate permission level
- [ ] If `EscalationThreshold = Executive Brief`: confirm the executive recipient of escalation emails is on the SharePoint access list before the next flow run
- [ ] Inform all authorized users of their obligations: do not forward P&C materials, do not discuss with unauthorized parties
