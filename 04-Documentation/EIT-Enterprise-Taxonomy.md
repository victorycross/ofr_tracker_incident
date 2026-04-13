# Enterprise Incident Tracker — Enterprise Taxonomy Reference

**Version:** 1.0
**Date:** February 2026
**Classification:** Internal

This document defines the authoritative taxonomy used across the Enterprise Incident Tracker. All domain leads, incident owners, and triage reviewers must apply these definitions consistently. Inconsistent use of taxonomy fields undermines reporting accuracy and escalation automation.

---

## 1. Domains

The EIT organises all incidents under one of five enterprise domains. Domain assignment determines which domain lead receives escalation notifications and which team is responsible for incident resolution.

| Domain | Abbreviation | Scope |
|--------|-------------|-------|
| **National Security** | NS | Incidents involving classified information handling, national security clearance breaches, insider threats within security-sensitive practices, or government-mandated reporting obligations related to national security matters |
| **NIS** | NIS | Network and information security incidents: cybersecurity threats, data breaches, phishing campaigns, malware, vulnerability exposures, DLP alerts, and technology-related information security events |
| **Privacy & Technology** | PT | Privacy regulatory incidents (PIPEDA, provincial privacy law), client data handling failures, AI tool misuse involving personal or confidential data, technology policy violations, and regulatory breach notification assessments |
| **Human Capital** | HC | Personnel incidents: harassment complaints, workplace conduct matters, employment litigation, employee performance investigations, and incidents requiring HR or CHRO involvement |
| **Physical Security** | PS | Physical security incidents at firm premises: unauthorized access, forced entry, theft, security system failures, physical threats to personnel, and incidents requiring police notification or physical safety response |

### Domain Assignment Rules

1. **Single domain per incident.** If an incident spans multiple domains (e.g., a cybersecurity breach that also involves personnel), assign the domain of the **primary impact or escalation authority**. Document the cross-domain nature in the `NextAction` field and use `RelatedItemIDs` to link any related incidents in other domains.
2. **Domain does not change after promotion.** Domain is set at intake and confirmed at triage. Changes after promotion require domain lead approval and a history entry documenting the reason.
3. **SubDomain field** is free-text and can be used to record more specific categorisation within a domain (e.g., `SubDomain = "PIPEDA s.10"` under Privacy & Technology, or `SubDomain = "Insider Threat"` under National Security).

---

## 2. Incident Types

IncidentType provides a second-level classification within a domain. The same type may apply across multiple domains.

| IncidentType | Definition | Primary Domains |
|-------------|-----------|-----------------|
| **Breach** | Confirmed or suspected unauthorized disclosure, access to, or loss of confidential, personal, or classified information | NIS, Privacy & Technology, National Security |
| **Vulnerability** | An identified weakness in systems, processes, or controls that has not yet been exploited but presents material risk | NIS, Privacy & Technology |
| **Policy Violation** | A confirmed failure to comply with an internal firm policy, regulatory requirement, or professional conduct standard | All domains |
| **Threat** | A credible or active threat requiring assessment or response — including insider threats, external threat actors, and physical threats to personnel | National Security, NIS, Physical Security |
| **Operational** | An incident arising from operational failure — process breakdown, system outage, or service disruption — that creates material risk | All domains |
| **Personnel** | An incident directly involving one or more named individuals in a conduct, performance, or employment context | Human Capital |
| **Physical** | A physical security event at a firm location: unauthorized access, forced entry, theft, or threat to physical safety | Physical Security |

### IncidentType Assignment Notes

- **Breach vs. Vulnerability:** A breach has been confirmed or is reasonably suspected to have occurred. A vulnerability is a risk that has not yet been exploited. Promote from Vulnerability to Breach if exploitation is confirmed during the incident lifecycle (document the change in history).
- **Threat vs. Breach:** A threat is forward-looking (the attack has not occurred). A breach is retrospective (the event has occurred). If a threat actor successfully executed their threat, reclassify as Breach and document the change.
- **Personnel incidents** must always be assigned to Human Capital domain. If a personnel incident also triggers a Policy Violation within another domain, create a linked incident in that domain via `RelatedItemIDs`.

---

## 3. Severity (ERM-Aligned, 5-Tier)

Severity is an enterprise risk management (ERM) classification reflecting the potential impact of the incident. It is assigned by the triage reviewer or domain lead — not inherited from the submitter's Priority field. Severity may be updated as the incident evolves.

| Severity | Color | Definition | Response Expectation |
|----------|-------|-----------|----------------------|
| **Critical** | Dark Red `RGBA(119,40,32,1)` | Enterprise-threatening. Potential for material regulatory sanction, significant client harm, national security compromise, or reputational damage requiring executive and board-level attention. | Immediate response. Domain lead notified within 2 hours. Executive Brief escalation threshold applies after 14 days without containment. |
| **High** | Red `RGBA(224,48,30,1)` | Significant impact on firm operations, client trust, regulatory standing, or employee welfare. Requires urgent triage and active management. | Domain lead engaged within 24 hours. Escalate threshold applies after 7 days without material progress. |
| **Moderate** | Orange `RGBA(228,92,43,1)` | Meaningful risk with defined scope. Does not pose immediate enterprise-level threat but requires timely resolution. Default severity on promotion — triage reviewer must confirm or adjust. | Active management required. Domain lead informed. Review at each status update. |
| **Low** | Blue `RGBA(65,83,133,1)` | Limited impact. Risk is contained or the incident involves a minor policy deviation with no immediate harm. Monitor and resolve within normal operational timelines. | Monitoring cadence. No escalation trigger. |
| **Informational** | Grey `RGBA(150,150,150,1)` | No immediate risk. Raised for awareness or record-keeping. May be a near-miss, a potential vulnerability that has been assessed as low likelihood, or a completed advisory. | Record and close when appropriate. |

### Severity vs. Priority

| Field | Set By | Purpose | When Updated |
|-------|--------|---------|--------------|
| **Priority** (3-tier: High/Medium/Low) | Submitter at intake | Operational triage urgency — how quickly the triage reviewer should look at it | Set at submission; does not change after promotion |
| **Severity** (5-tier ERM) | Triage reviewer / domain lead | ERM impact classification — drives escalation thresholds and leadership reporting | Set at triage; updated as the incident evolves |

These are independent fields. A Low priority intake item may become a Critical severity incident after triage assessment. Do not treat them as equivalent.

---

## 4. Status

Status represents the current lifecycle state of the incident. Status drives Kanban column placement and is recorded in every history entry.

| Status | Definition | Typical Transition |
|--------|-----------|-------------------|
| **New** | Incident has been promoted from intake. Triage is underway but the incident has not yet been actively worked. | New → Active (when domain lead takes ownership and work begins) |
| **Active** | Actively being managed. Investigation, remediation, or containment efforts are in progress. | Active → Escalated, Monitoring, Contained, or Closed |
| **Monitoring** | Immediate response phase is complete; the incident is in an observation period to confirm resolution or detect recurrence. | Monitoring → Contained or Closed |
| **Escalated** | Formally escalated to the domain lead or executive sponsor. Active management continues in parallel. | Escalated → Active (de-escalated), Contained, or Closed |
| **Contained** | The incident has been contained — the immediate risk has been neutralised. Documentation, lessons learned, and post-incident review may still be in progress. `DateContained` is recorded at this transition. | Contained → Closed |
| **Closed** | Fully resolved. All actions complete, documentation finalised, and the incident is archived. Closed incidents are excluded from all active views and KPIs. | Terminal state — only reopened by creating a new linked incident |

### Status Rules

1. **Closed is final.** Do not reopen closed incidents. If a closed incident recurs or has new information, create a new incident and link it via `RelatedItemIDs`.
2. **Contained ≠ Closed.** An incident should be marked Contained when the immediate risk is neutralised but post-incident work remains. Close only when all work is complete.
3. **Status changes must be documented.** Every status change requires a history entry via the Add Update form. The `StatusAtUpdate` field in `EIT_IncidentHistory` records the new status at the time of the update.

---

## 5. Escalation Threshold

EscalationThreshold is set automatically by the Daily Staleness & Escalation Calculator flow based on Severity and days since last update. It can also be set manually via the IssueDetailScreen Add Update form (UpdateType = Escalation).

| EscalationThreshold | Color | Meaning | Auto-Set Condition |
|--------------------|-------|---------|-------------------|
| **Not Escalated** | Grey | No escalation has been triggered. Normal monitoring. | Default on promotion |
| **Watch** | Grey | The incident is approaching escalation thresholds. Domain lead is advised to review. | *(manual only — reserved for domain lead use)* |
| **Escalate** | Red | Formally escalated to the domain lead. Escalation notification email sent to `EIT_DomainContacts.LeadEmail`. | Severity=High AND DaysSinceUpdate>7 |
| **Executive Brief** | Dark Red | Escalated to executive sponsor. Executive Brief email sent to `EIT_DomainContacts.EscalationEmail`. | Severity=Critical AND DaysSinceUpdate>14 |

> **See:** `04-Documentation/EIT-Escalation-Thresholds.md` for the full escalation matrix and manual override procedures.

---

## 6. Confidentiality Level

ConfidentialityLevel is an information-handling classification. It does not restrict SharePoint access — it is an informational label for how incident information should be handled.

| Level | Definition |
|-------|-----------|
| **Internal** | Default. Visible to all `EIT Members`. Information should not be shared outside the firm. |
| **Restricted** | Sensitive information requiring discretion. Share only on a need-to-know basis within `EIT Members`. |
| **Confidential** | Highly sensitive. Share only with directly involved parties. Must be set when `PrivilegedAndConfidential = Yes`. |

> **ConfidentialityLevel vs. PrivilegedAndConfidential:** Confidentiality level is an information-handling label. P&C designation is a legal status that restricts SharePoint access via item-level permissions. An incident can be `Confidential` without being P&C. An incident designated P&C must also be set to `Confidential`. See `EIT-Access-Control-Guide.md`.

---

## 7. Source System

SourceSystem records where the incident record originated.

| Value | Meaning |
|-------|---------|
| **Power Apps** | Submitted via the EIT canvas app intake form (default) |
| **ServiceNow** | Record created or imported from ServiceNow (Phase 2 integration) |
| **iSight** | Record created or imported from iSight (NIS threat intelligence platform) |
| **Manual Entry** | Record created directly in the SharePoint list by an EIT administrator |

---

## 8. Update Type

UpdateType classifies each history entry in `EIT_IncidentHistory`.

| UpdateType | When to Use |
|-----------|-------------|
| **Status Change** | The incident's status changed (e.g., New → Active, Active → Contained) |
| **Evidence Added** | New evidence, documents, or findings have been documented |
| **Escalation** | A formal escalation decision was made — triggers the Domain Escalation Notifier flow |
| **Containment** | The incident has been contained — use with Status = Contained |
| **Closure** | The incident is being closed — use with Status = Closed |
