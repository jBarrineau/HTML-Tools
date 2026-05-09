# Offboarding Power Platform Developers

A guide for teams without admin privileges covering inventory, handover, access removal, and automation of the offboarding process.

---

## Overview

When a Power Platform developer leaves a team, three things must happen cleanly:

1. All apps, flows, connectors, and reports they own must be identified and transferred to new owners.
2. All personal connections they hold must be migrated to service accounts before their account is disabled.
3. Their access to Teams, Groups, SharePoint, environments, and licenses must be revoked.

This document covers how to manage this process without admin privileges, which tools and APIs can automate the inventory step, and how to structure the overall tooling in Power Platform.

---

## Tool Design Principles

Because most team members are not Power Platform or M365 admins, the offboarding tool must:

- **Separate tasks by who can do them** — self-service (team lead), developer-driven (the departing person), and admin-required (IT/helpdesk).
- **Generate admin requests automatically** — rather than expecting non-admins to navigate admin portals, the tool produces a ready-to-send email listing every admin action required.
- **Track completion** — each task has a status that can be ticked off as confirmed, giving a clear audit trail.
- **Be built in-platform** — a Canvas App backed by a SharePoint list keeps everything in the team's existing stack, with Power Automate flows handling notifications and email generation.

---

## Recommended Build Stack

| Layer | Technology | Purpose |
|---|---|---|
| UI | Canvas App | Checklist, assignment form, progress tracking |
| Data | SharePoint List | One row per offboarding, columns per task |
| Automation | Power Automate | Inventory queries, admin email, owner notifications |
| Inventory | Power Platform API / Power BI REST API | Automated asset discovery (see Phase 1) |

---

## The Four-Phase Checklist

### Phase 1 — Inventory

The goal is to produce a complete list of everything the departing developer owns before their last day. Tasks are split by what can be done programmatically (see the Automation section below) and what must be gathered manually.

| # | Task | Who | Method |
|---|---|---|---|
| 1.1 | List all Canvas Apps owned | Dev + Admin | Power Automate flow (see automation) |
| 1.2 | List all Cloud Flows owned | Dev + Admin | Power Automate flow (see automation) |
| 1.3 | List all custom connectors created | Dev | Power Automate flow (maker level) |
| 1.4 | List Power BI reports and datasets | Dev | Power BI REST API (maker level) |
| 1.5 | Identify connections using personal credentials | Dev + Admin | Dev lists own; admin lists all |
| 1.6 | Agree new owner for each asset | Team lead | Canvas App assignment form |

### Phase 2 — Power Platform Handover (Developer Does)

These actions should be completed by the departing developer themselves, ideally in the final week of their notice period.

- Share all Canvas Apps with the new owner (co-owner role) via the Power Apps maker portal.
- Add new owner as co-owner on personal Cloud Flows via My Flows > Share.
- Export standalone (non-solution) flows as `.zip` backup files.
- Export non-solution Canvas Apps as `.msapp` backup files.
- Add the new Power BI workspace admin before leaving (Workspace Settings > Access).
- Document any hardcoded email addresses or UPNs in apps and flows so the new owner can update them.
- Run a knowledge transfer session and save notes to a shared location.

### Phase 3 — Admin Actions (Request via Generated Email)

These actions require Power Platform Admin or M365 Admin access. The offboarding tool generates a pre-populated email covering all of the following.

**Power Platform**
- Reassign Canvas App ownership in the Power Platform admin center.
- Reassign Cloud Flow ownership (admin center or PowerShell).
- Migrate personal connections to service account connections — **must happen before the account is disabled**.
- Remove the developer from all environment security roles (Maker, Admin, etc.).
- Rotate or re-key any API keys or secrets they managed.

**Microsoft 365 / Entra ID**
- Remove from all Teams channels and Microsoft 365 Groups.
- Remove from SharePoint site ownership and membership.
- Remove from distribution lists and mail-enabled groups.
- Remove from Azure DevOps or GitHub repos used for ALM pipelines.
- Revoke Power Apps / Power Automate per-user licences (so they can be reassigned).
- Disable Entra ID account on last day (disable before delete — provides a buffer).
- Delete or convert the account after a 30-day hold period.

### Phase 4 — Verify and Sign Off

- Confirm all apps still function correctly under new ownership.
- Check flow run history — confirm no flows are failing after ownership transfer.
- New owners confirm they can access and edit all transferred assets.
- Confirm admin has completed all requested actions.
- Sign off checklist with team lead.

---

## Critical Timing Note

> **The single most common cause of post-offboarding failures is personal connection migration.**
>
> Flows that use a departing developer's personal OAuth connection (SharePoint, Teams, Outlook, Dataverse, etc.) will **start failing immediately** when their account is disabled. This migration must be explicitly flagged as urgent in the admin request, and completed **at least one working day before** the account is disabled — not on the last day.

---

## Phase 1 Automation — What Can Be Done Programmatically

### Summary

| Inventory Item | Automatable? | Permission Level | Method |
|---|---|---|---|
| List Canvas Apps | Partial | Maker (own apps) + Admin (all envs) | Power Automate + REST API |
| List Cloud Flows | Partial | Maker (non-solution) + Admin (solution flows) | Power Automate + REST API |
| List Custom Connectors | Yes — fully | Maker only | Power Automate (maker connector) |
| List Power BI reports / datasets | Yes — fully | Maker only | Power BI REST API |
| List personal connections | Admin required | Admin for cross-user view | Power Apps Admin connector |
| Assign new owners | Yes — fully | Maker only | Canvas App + SharePoint |

---

### 1. Canvas Apps — Partial Automation

**Maker level (own apps only)**

The departing developer runs a Power Automate flow using the **Power Apps for Makers** connector. The action `Get apps as the signed-in user` returns all Canvas Apps they own in a given environment. This must be looped across all environments.

**Admin level (all environments)**

```
GET https://api.powerplatform.com/powerapps/environments/{environmentId}/apps
    ?api-version=2022-03-01-preview
```

Requires Power Platform Admin role. Returns all apps with owner principal details, filterable by owner UPN.

> **Caveat:** The maker connector is scoped to one environment per call and only returns the signed-in user's apps. The admin endpoint covers all environments but requires elevated permissions.

---

### 2. Cloud Flows — Partial Automation

**Maker level (non-solution flows only)**

Use the **Power Automate Management** connector action `List my flows`, looped per environment. Returns all standalone flows owned by the signed-in user.

**Admin level (includes solution-aware flows)**

```
GET https://api.powerplatform.com/powerautomate/environments/{environmentId}/cloudFlows
    ?ownerId={userId}&api-version=2022-03-01-preview
```

> **Caveat:** This is a known platform limitation. The admin API endpoint does not return solution-aware flows when called without the `includeSolutionCloudFlows` parameter, and that parameter is not supported on the admin endpoint. To capture solution flows, use the admin REST API with the `ownerId` filter. Both calls are needed for a complete picture.

---

### 3. Custom Connectors — Fully Automatable (Maker)

Use the **Power Apps for Makers** connector: `Get custom connectors as the signed-in user`. The departing developer can run this themselves.

```
GET https://api.powerapps.com/providers/Microsoft.PowerApps/apis
    ?api-version=2016-11-01
    &$filter=environment eq '{environmentName}' and isCustomApi eq true
```

> **Caveat:** Custom connectors inside solutions are not returned by `Get-AdminPowerAppConnector`. Check solution contents separately via the Dataverse API if needed.

---

### 4. Power BI Reports and Datasets — Fully Automatable (Maker)

Call the Power BI REST API using a delegated token (the departing developer's account). No admin role required.

```
# Step 1 — list all workspaces the user belongs to
GET https://api.powerbi.com/v1.0/myorg/groups

# Step 2 — for each workspace, list reports
GET https://api.powerbi.com/v1.0/myorg/groups/{groupId}/reports

# Step 3 — for each workspace, list datasets
GET https://api.powerbi.com/v1.0/myorg/groups/{groupId}/datasets

# Step 4 — items in personal "My Workspace"
GET https://api.powerbi.com/v1.0/myorg/reports
GET https://api.powerbi.com/v1.0/myorg/datasets
```

> **Caveat:** Only items in workspaces the user is a member of are returned. An admin-level call using `GET /v1.0/myorg/admin/reports` is required to find any orphaned items across the full tenant.

---

### 5. Personal Connections — Admin Required

The departing developer can list their own connections using the **Power Automate Management** connector: `List my connections`. This returns connector type, status, and last modified date — useful for self-reporting.

To see all connections across the environment and identify flow-level risk:

```
GET https://api.powerapps.com/providers/Microsoft.PowerApps/scopes/admin/environments/{environmentId}/connections
    ?api-version=2016-11-01
```

Requires Power Platform Admin role. Connection credentials are never exposed — only metadata.

---

### 6. Owner Assignment — Fully Automatable (Maker)

Once the inventory queries populate a SharePoint list, a Canvas App screen with a People Picker control lets the team lead assign a new owner to each asset. A Power Automate flow then notifies the new owner to confirm acceptance. No admin access required.

---

## Suggested Power Automate Flow Architecture

```
Trigger: SharePoint item created (new offboarding record)
    │
    ├─► Pass 1 — Delegated token (run as departing developer)
    │       List my flows (per environment)
    │       Get apps as signed-in user (per environment)
    │       Get custom connectors (per environment)
    │       List my connections
    │       Power BI: Get groups → Get reports/datasets
    │       Write results → SharePoint Inventory list
    │
    ├─► Pass 2 — Admin token (service account or admin-run)
    │       List cloud flows (ownerId filter, all environments)
    │       Get admin apps (all environments)
    │       Get connections as admin (flag personal creds)
    │       Write gaps → SharePoint Inventory list
    │
    └─► Generate admin request email
            Compose email from SharePoint task list
            Send to IT helpdesk / admin contact
```

---

## Canvas App Structure

The Canvas App should have three screens:

1. **Setup screen** — enter departing developer name, last day, offboarding lead, and admin contact. Creates the SharePoint record and triggers the inventory flow.
2. **Checklist screen** — four-phase task list. Self-service and developer tasks are interactive checkboxes. Admin tasks show as locked with a status indicator (pending / confirmed by admin).
3. **Admin request screen** — shows all outstanding admin tasks, allows selection of which to include, and displays the generated request email ready to copy or send.

---

## SharePoint List Schema

Suggested columns for the offboarding tracking list:

| Column | Type | Notes |
|---|---|---|
| Title | Single line | Departing developer name |
| LastDay | Date | Last working day |
| OffboardingLead | Person | Team member managing the process |
| AdminContact | Single line | IT helpdesk name / email |
| Status | Choice | Not Started / In Progress / Complete |
| Phase1Complete | Yes/No | Inventory done |
| Phase2Complete | Yes/No | Developer handover done |
| Phase3Complete | Yes/No | Admin actions confirmed |
| Phase4Complete | Yes/No | Verified and signed off |
| InventoryItems | Multi-line | JSON blob from automated inventory flow |
| AdminRequestSent | Yes/No | Email generated and sent |
| AdminRequestDate | Date | When request was sent |
| Notes | Multi-line | Free text for edge cases |

---

## References

- [Power Platform API — Get Admin Apps](https://learn.microsoft.com/en-us/rest/api/power-platform/powerapps/apps/get-admin-apps)
- [Power Platform API — List Cloud Flows](https://learn.microsoft.com/en-us/rest/api/power-platform/powerautomate/cloud-flows/list-cloud-flows)
- [Power Automate Management Connector](https://learn.microsoft.com/en-us/connectors/flowmanagement/)
- [Power BI REST API — Reports](https://learn.microsoft.com/en-us/rest/api/power-bi/reports)
- [Power BI REST API — Datasets](https://learn.microsoft.com/en-us/rest/api/power-bi/datasets/get-datasets)
- [PowerShell for Power Platform Administrators](https://learn.microsoft.com/en-us/power-platform/admin/powerapps-powershell)
- [Power Platform Inventory (Admin Center)](https://learn.microsoft.com/en-us/power-platform/admin/power-platform-inventory)
- [Manage connections in Power Automate](https://learn.microsoft.com/en-us/power-automate/add-manage-connections)

---

*Document prepared for internal team use. Last updated April 2026.*
