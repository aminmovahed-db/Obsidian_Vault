---
type: area
tags:
  - alinta
  - customer
created: 2026-03-17
---

# Alinta Energy

## Key Contacts

| Name | Role |
|---|---|
| Nick | CIO — 8 GMs, Brad is one |
| Brad | GM of Data — owns the platform |
| Jemma | GM for Data — owns analytics and AI |
| Jack Josh Elliot | Head of Data Engineering |
| Dave | Head of DevOps — admin of the platform |
| Andrew | Delivery Lead / PM |
| Gavin (Gavin Chue) | Governance Lead |
| Souhrab | SA |
| Gavin | AE |
| Billy O'Connor | Previous engagement (context on DevOps/Terraform) |
| Chuong | Previous engagement (context on approvals) |
| Allison | Reviewed Alinta's Terraform repo |
| Erik | Internal — standup partner |
| Armon | Internal — standup partner, workshop agenda owner |
| Jake | Internal — available for context |
| Beyza | Internal — design consultation (with Billy, Chuong, Allison) |
| Link | Technical (Alinta) — alongside Jake |
| Girish | Internal — communications loop (Erik, Girish, Armon) |

**Help desk**: 1300 664 343
**Best meeting days**: Mondays and Fridays
**Technical contacts**: Dave, Jack, Josh

**Internal standup**: Monday & Wednesday with Erik and Armon

## Partner

- Fero/Feuro (name to be confirmed) — DevOps partner
- Fuji — Development partner

## Project: Pilot for Teams

Structure for teams outside DnA. Each team gets their own catalog, schemas, assets, access to apps, Genie rooms, etc.

**Environment setup:**
- Non-prod: read-only access to production data (for model building)
- UAT catalog replicated from UAT env
- Test UAT before going to prod

**User types**: Business users, dev users, UAT users, prod users
**DnA work**: Pipelines, repos for pipelines and users

### Priorities
1. **Workspace and catalog** — must have
2. **Tagging** — second priority (automated tagging, taxonomy, access expires after 90 days)
3. **Automation** — depends on tagging completion
4. **Path to production** — de-scoped

### Challenge
Automation depends on tagging. If tagging isn't finished soon, the work needs to be done by the partner or in-house. Investigating if Databricks can do the build.

### Budget
No extra budget — proceeding with PS as far as budget allows. Partner expected to do development.

## Key Documents

- [Final Presentation Deck](https://docs.google.com/presentation/d/1Jd8ZDHcErYeLiRxl8SQoS4Gvt-GvY6kyLQd6Y6Mgqs8/edit?slide=id.g380036ad87e_0_282#slide=id.g380036ad87e_0_282)
- [Final Solution Design](https://docs.google.com/document/d/1xHKisvagYV7-4s3_tnhAwhiHCPhIBGoVr5cthrnb9mU/edit?tab=t.0)
- [Allison Historical Notes 1](https://docs.google.com/document/d/1gGS1Q41fcjDvoMQe6yz9cl3ak5kVcduM6E_0jWGtdlc/edit?usp=sharing)
- [Allison Historical Notes 2](https://docs.google.com/document/d/1hCpTxLaQbCkH7d5p2J0ulatSxhT4DZq5Gu9bJy0xraM/edit?usp=sharing)
- [[Alinta Detailed Design - Catalog and Security]]

## Implementation Plan

![[Attachments/Pasted image 20260317131616.png]]

![[Attachments/Pasted image 20260317151318.png]]

## Kick Off Prep

- [ ] Email account
- [ ] Calendars
- [ ] Jira
- [ ] Teams

No kick off pack exists. Pre-partner engagement with Feuro.

## ADH Platform

- ADH = Alinta Data Hub (Alinta's data platform)
- Users: DnA users and non-DnA users
- DnA has tight control of everything — user workspace and pipeline workspace, prod user to dev user
- They want more governance via a dev user workspace

## Unity Catalog Architecture

### Current State
- Single 'Pilot' catalog shared across teams
- 1 Pilot catalog → multiple schemas (challenging to manage)

### Recommendation
Transition to domain-based architecture: each team gets a dedicated catalog (e.g., `pilot_retail`, `pilot_finance`). Provides better data isolation, simplifies access control, and enhances scalability without requiring changes to existing Entra AD groups.

**Structure**: Business unit → catalog, project → schema. New business units get a new catalog.

### Status
- Andrew is the decision maker
- DnA has already approved the catalog design but not the rest of the teams
- Per Billy, other teams may have already accepted — needs confirmation with Chuong

### Next Steps
1. Formally align on and define the data domains that will map to the new catalogs
2. Adjust Terraform scripts to provision new teams with a dedicated catalog instead of a schema
3. Plan migration of existing assets to the new domain catalogs, initially preserving schema names to minimize disruption

### Design Document Scope
- Depends on existing DevOps process
- Need access to their Terraform code to include changes in the design
- Need access to their workspace to review catalog structure
- Billy noted previous recordings/meetings are lost due to access expiring; David was the expert, Allison reviewed the Terraform repo

## Open Questions

- [x] Confirm partner name (Feru/Feuro/Fero?)
- [ ] Can Databricks do the build? What is the effort?
- [ ] DnA workshop prep for week starting 23 March
- [x] Confirm with Chuong that non-DnA teams have accepted the catalog design (done)
- [x] Get access to Alinta's Terraform repo and workspace (done)
- [x] What should be included in the UC design document scope?
- [ ] Ask Andrew where to put design documents (via Wednesday meeting)
- [ ] ABAC roles and row/column-level masking — to be considered in security design
- [ ] Confirm what teams currently have in their catalogs (tables, models, volumes) before workshop

## DASDR Process

- DASDR = DnA Architecture and Solution Design Review
- Design review board for key decisions
- Submission on Tuesday → Presentation on Thursday → Approval
- For now, DASDR is **NOT** necessary for this engagement

## UC Detailed Design Scope

Scope confirmed with Chuong/Erik (2026-03-19):

1. Naming Standards
2. Catalogs
3. Default Schemas
4. Volumes
5. Permissions / Grants

**Key clarifications:**
- Terraform build is a separate task (not in this scope)
- Security includes ABAC roles, row/column-level masking, models, everything under each catalog
- Need to understand what teams already have in their catalogs

## Key Dates

- **13 April 2026** — Start date for Unity Catalog implementation
- **27 April 2026** — Start date for Tagging implementation

## Forwarded Emails (from Allison)

Reference list of emails tagged "Alinta" for historical context:

- Pilot for Teams - Data Science Notes
- Pilot for Teams Uplift - DnA Requirements
- Pilot for Teams Uplift - Merchant Tech
- Pilot for Teams Uplift - Digital Follow-up notes
- Pilot for Teams Uplift - ML Ops Walkthrough

## Meeting Log

- 2026-03-17 — First meeting and follow-up
- 2026-03-18 — Workshop planning, UC architecture discussion, Billy O'Connor context on DevOps/Terraform
- 2026-03-19 — Scoping call with Chuong and Erik. Confirmed UC design scope (catalogs, schemas, volumes, permissions). DASDR not required. Jake/Link are technical contacts. Beyza to help with design. Erik away — Monday meeting to be moved to Wednesday. Key dates: UC implementation 13 April, Tagging 27 April. Allison shared historical docs and forwarded emails.
