---
source: https://docs.google.com/presentation/d/1Jd8ZDHcErYeLiRxl8SQoS4Gvt-GvY6kyLQd6Y6Mgqs8/edit
type: google-slides
captured: 2026-03-21
description: Alinta Energy Unity Catalog final presentation — 78 slides covering catalog architecture, security model, team permissions, and automation
---

## Slide 1 — Slide 1

*Pilot for Teams Design*
*Alinta Energy DnA*
©2025 Databricks Inc. — All rights reserved
Billy O’Connor, Senior Solutions Consultant, DatabricksAllison Chau, Resident Solution Architect, Databricks
Dec 2025

## Slide 2 — Final walkthrough

Done:
General - Update any reference to a “Con” and rebrand as consideration; only call it a Con if it is actually a negative or a cost.
**- Done**
Is there anything we have left on the table with this solution design that DnA team needs to be aware of i.e any best practices that havent been aligned to ? Lock down the connectivity to the workspace ?
People can call through external API and no control -
**Addressing it through network policies and permissions in Solution Doc**
Production is for DnA team to “support”, as well as monitoring - can we call out in slides and word doc please -
**Addressed during meeting, covered in steps 15 & 16**
Confirm who is able to grant access to Workspace, catalog, and items. - ensure it’s clear in doc and these slides.
And what Databricks capability can Pilot teams use –
**Confirmed by step 6 pilot_uat, with the pilot team responsible for assigning pilot UAT testers.**
Pilot manager to provide access to the UAT assets - administrator for the pilot / responsible for the UAT asset (calling this out in sol doc or slide if possible) -
**DnA responsible for Business UAT testers (Step 13a)**
Step 8 - is more than testing - it’s consumption of non-prod assets by the business - can we call out in slides and word doc please -
**Data Assets will be covered by Unity Catalog, other assets see section 5.5 in the Solution Design Document**
No Action Required::
30/60/90 day rule for UAT lifecycle (lockout to cleanup) -
**Controlled by LCM rules**
Persona names may need to be updated in their change management docs -
**Action on Alinta**
Acceptable asset types for each of the personas and acceptable sourcing / security of the workspace
**- High level guidance provided (in requirement 4 and Solution Design doc), further refinement to be actioned by Alinta if required.**
As part of this - for ML path to production, Brad indicated that “feature development” is an engineering activity for DnA; and inference and consumption of models is Data Science.
**- Just a consideration by Andrew, not an actual action**
*INTERNAL - FOR TRACKING OF ITEMS RAISED 12 JAN 2026*

## Slide 3 — Pilot for Teams

The Pilot for Teams initiative aims to
democratise data access
and enable teams across Alinta to use the ADH platform as their primary platform for data manipulation, management, prototyping, and analytics.
The goal is to strike a balance between empowering users with liberal access to tools and
maintaining governance
and cost control to avoid unregulated growth of pseudo-production assets.
In addition, the Pilot for Teams design needs to provide a
pathway to production
for those solutions require ongoing operations and management.
**Data Accessibility**
- Self-service data exploration
**Controlled Collaboration**
- Secure, shared workspaces
**Pathway to Production**
- Sandbox to production pipeline
**User Segmentation**
- Tailored access and permissions by 3 identified roles
**Governance and Monitoring**
- Auditable, managed environment
**Automation**
- Orchestrated, scalable
**Regulatory Compliance**
- Aligned to FIRB requirements
**Cost Management**
- Clear cost accountability
Recap of Objectives and Requirements
*Objectives*
*Pilot for Teams Requirements*

## Slide 4 — Objectives and our Recommended Solution to your Requirements

The Pilot for Teams initiative aims to
democratize data access
and enable teams across Alinta to use the ADH platform as their primary platform for data manipulation, management, prototyping, and analytics.
The goal is to strike a balance between
empowering users
with liberal access to tools and
maintaining governance
and control to avoid unregulated growth of pseudo-production assets.
This initiative has been grounded on agreed requirements, which have formed the foundation of Databricks’ recommendations and proposed design.

|  | Focus Area / Requirement | Databricks Recommended Solution | Alinta Review Status [Status as on 18 Dec] |
|---|---|---|---|
| 1 | Data Accessibility - Self-service data exploration | Scaled Governance Solutions | [1 Dec]Agreed next steps: Billy and Beyza to connect and review and establish whether the recommended solution applies or is redundant based on new changes advised by Alinta (Andrew) (access given to groups as per current process stays) |
| 2 | Controlled Collaboration - Secure, shared workspaces | Workspace / Catalog Design | [5 Dec]Solution recommendation discussed and APPROVAL gained from Alinta team (Andrew, David and Beyza), with some minor feedback to rename the slide to Catalog & Workspace Redesign |
| 3 | Pathway to Production - Sandbox to production pipeline | Pathway to Production | [Dec 5th outcome]We secured your review and APPROVAL for the recommended solution design on SQL Analyst + Data Scientist personas onlyTo DO on 12th Dec:- request to reschedule to 18 Dec[18 Dec]All 3 personas (SQL Analyst + Data Scientist +  IT developer) discussed, with some minor feedback change to the slides and update back to the Design doc. Agreed of Databricks todo change and send to Andrew for offline review. Overall agreement gained from Andrew + Beyza and David. Agreed to setup session with Brad in Jan 2026 for a final sign off. |
| 4 | User Segmentation - Tailored access and permissions by 3 identified roles | Workspace / Catalog Design + Pathway to Production | [18 Dec]NO major feedback change captured, with an overall agreement gained from Andrew + Beyza and David. Agreed to setup session with Brad in Jan 2026 for a final sign off. |
| 5 | Governance and Monitoring - Auditable, managed environment | Tagging | [1 Dec]Recommendation discussed andAPPROVALgained from Alinta team. |
| 6 | Automation - Orchestrated, scalable | Automation | [1 Dec]Recommendation discussed andAPPROVALgained from Alinta team. |
| 7 | Regulatory Compliance - Aligned to FIRB requirements | Tagging & Automation | [1 Dec]Recommendation discussed andAPPROVALgained from Alinta team.Alinta to further internally explore archiving process for any exception requests (dealt by Click Ops process) |
| 8 | Cost Management - Clear cost accountability | Tagging | [1 Dec]Recommendation discussed andAPPROVALgained from Alinta team. |

## Slide 5 — Current State Architecture

## Slide 6 — Future State: A Pilot Environment

**Pilot Environment (DEV-User)**
Data
**Interactive Cluster:**
Pilot_<team>
Adhoc Queries
**External Data:**
Databases/APIs/Tools /SFTP
**Lakehouse Federation:**
Databases that we decide to enable direct access to
**Databricks Secrets**
**UAT Environment (UAT-User)**
Environment dedicated to
**Pilot for Teams development**
Data
**Jobs:**
Scheduling of active pilot work – cleaned up after pilot is completed
Environment dedicated to
**User Acceptance Testing**
team_volume
**Shared Interactive Cluster**
**SQL Warehouse**
prod data for comparison to uat data
**PROD Environment (PROD-User)**
Data
Environment dedicated to Prod Data
**Consumption**
**Shared Interactive Cluster**
**SQL Warehouse**

## Slide 7 — End to end process

DnA
Team
Pilot
Team
Automation
**Provisioning**
**Production**
**(PROD-User)**
UAT
Team
**Building**
**(DEV-User)**
**Pilot UAT**
**(UAT-User)**
**Business UAT**
**(UAT-User)**
**Deployment**
Approve
UAT team access to
**uat**
**13a**

## Slide 8 — Requirement 1 : Data Accessibility - Self-service data exploration

Recommended Solution: Scaled Governance Solutions
Users need a way to manipulate and analyze data with Databricks using SQL and Python. The D&A team want to minimize any dependencies on local storage or third party solutions for data management.
**Our Understanding:**
Users need a way to access the tools they need, and ideally should not need to leave the Databricks Platform in doing so. Their access to these tools should be provided in a way that reduces time to market for data products.
*Requirement*
First half of the requirement is addressed by the Databricks platform out of the box, so our solution focuses removing friction in this process.
Catalog per team designs grants Pilot team some flexibility whilst retaining control.
Managed Volumes - Provides governed storage for unstructured files entirely within Databricks, removing need for Azure storage containers.
Provisioned access to compute, workspaces and storage (Unchanged).
Databricks Secrets (Non Key-vault) to protect sensitive credentials.
*Recommended Solution*
**Pros:**
**Ease of use increases development velocity.**
**Removes need for Azure portal access.**
**Removes the ability to bypass of Unity Catalog using External Volumes.**
**Considerations:**
**Tweaking of Terraform to provide provision catalog and managed volumes.**
**Some upfront effort to move to catalog approach.**

## Slide 9 — Future State - Scaled Governance

Implementing an entitlements framework
ServiceNow
ServiceNow Access Request
Unity Catalog
Databricks Workflows
Databricks App
Notebook task: enforce_entitlements.py
Team Catalogs, Schemas, Folders
operations.entitlements_table
Assignments / Revocations
Apply
Mark as closed
Retrieve
Approval Chain
Retrieve
Granted
Actual
PfT 360 Security Dashboard
1
2
3
4
5
**Agreed not to include - retained for reference**

## Slide 10 — Req 2 : Controlled Collaboration - Secure Shared Workspaces

Databricks Recommended Solution: Catalog & Workspace Redesign
Allow small teams to share data within their groups. Prevent broad sharing of shadow assets without governance.
**Our Understanding:**
Teams should be provisioned access to a collection of team datasets, and should be able to collaborate and access datasets as they are created by their team. A solution is required to prevent the ad-hoc, ungoverned sharing of datasets by Pilot Teams.
*Requirement*
Continue to provision teams with access to a shared collection of datasets, however tweak this to provision access to a team catalog instead of a shared schema.
Move development activities from ‘Prod User’ workspace to ‘Dev User’ workspace.
Move UAT activities to from ‘Prod User’ workspace to ‘UAT User’ workspace.
Leverage the pilot_UAT catalog as a destination for Pilot Teams to promote assets ready for UAT testing.
Enforce Catalog bindings so that users can only access assets from the appropriate workplaces.
Nightly job to remove manage privileges and prevent ad-hoc sharing.
*Recommended Solution*
**Pros:**
**Improve user experience (Developer and Business users) and setup the Databricks platform for greater scalability.**
**Workspaces are organized around a common use case, simplifying access, auditing and observability.**
**Simplifies pathway to production.**
**Considerations:**
**Effort required to move assets across workspaces and catalogs.**
**Further user education and communication required.**

## Slide 11 — Future State: A Pilot Environment

**Pilot Environment (DEV-User)**
Data
**Interactive Cluster:**
Pilot_<team>
Adhoc Queries
**External Data:**
Databases/APIs/Tools /SFTP
**Lakehouse Federation:**
Databases that we decide to enable direct access to
**Databricks Secrets**
**UAT Environment (UAT-User)**
Environment dedicated to
**Pilot for Teams development**
Data
**Jobs:**
Scheduling of active pilot work – cleaned up after pilot is completed
Environment dedicated to
**User Acceptance Testing**
team_volume
**Shared Interactive Cluster**
**SQL Warehouse**
prod data for comparison to uat data
**PROD Environment (PROD-User)**
Data
Environment dedicated to Prod Data
**Consumption**
**Shared Interactive Cluster**
**SQL Warehouse**

## Slide 12 — Future State - Scaled Governance

Alinta - Databricks Account
Pilot for Teams - Databricks Workspace
Team: CorpFinance
Compute
**Pilot Teams by Business Unit**
**Retail**
**CorpFinance		Retail**
**Customer Engagement	RetailCredit**
**DnA			RetailFinance**
**OpAssurance		TPMEast**
**Merchant**
**EnergySupplyTech	TPMWest**
**Corporate**
**RetailAssurance		RiskAnalytics**
UC Catalog: pilot_corp_finance
Policies
**+**
Terraform provisioning:
1. Provision assets
2. Apply security and grants: - Catalog
- Schema
- Volume
- Shared Cluster
- Warehouse
3. Apply standard tagging
Team Manager
User
SCIM provisioning
Entra group created
Tables
Row Filters and Column Masking
Schemas
Requests team provisioning
Requests access to team
E
ntra Team Group
E.g. GG-Azure-DataHub-Pilot-CorpFinance
User added to group
Shared Workspace Files location
Provisioning a Team
Team: Retail
Team: n+1
Team: Customer Engagement

## Slide 13 — Req 3: Pathway to Production - Sandbox to production

Databricks Recommended Solution: Pathway to Production
Enable a structured process for moving stable assets from pilot environments into production. Differentiate pathways for machine learning (ML) models versus traditional data engineering assets.
**Our Understanding:**
A DNA team priority is time-to-value. As such, they wish to streamline the process by which assets are moved from the Pilot for Teams environment to productionized data products. Any recommended solution should reflect that ML models will likely require a different path to production than traditional data engineering assets.
*Requirement*
Standardize development in the Dev User workspace, and promote to UAT by writing to the team’s UAT Schema.
Implement standards depending upon the skill level of each user type.
Enforce the following standards on these Pilot for Teams user types:
Developers: Utilize a feature branch, that incorporates dbt and DABs to promote through environments.
Business Analysts: A handoff checklist that includes: Business Justification, Working SQL Scripts, Source Tables, Expected Outputs, and key stakeholders.
Data Scientists: Parameterized Code using widgets, Upskilling in Pyspark, utilze DABs.
*Recommended Solution*
**Pros:**
**Reduces the time to transition data assets.**
**Shifts the burden of cross-environment parameterization onto pilot teams.**
**Assets have to pass a cross-catalog test just to make it to UAT.**
**Creates trail of responsibility as byproduct.**
**Considerations:**
**Minor upskilling required of Pilot for Teams users.**
**Development teams obliged to conform to DNA standard.**
**No**
**enforcement**
**of pyspark (continuation of existing process).**

## Slide 14 — Path to Production - SQL Analysts

**Pilot Environment**
**(DEV-User)**
Data
**UAT Environment (UAT-User)**
**PROD Environment (PROD-User)**
**DEV Pipeline**
**UAT Pipeline**
**PROD Pipeline**
Feature Branch
Main Branch
Jira
**Submit**
**Ticket**
**adh_data_model/adh_general**
Release Branch
**External Data**
**+ Lakehouse Federation**

## Slide 15 — Path to Production - Data Scientists

**Pilot Environment**
**(DEV-User)**
Data
**UAT Environment (UAT-User)**
**PROD Environment (PROD-User)**
**DEV Pipeline**
**UAT Pipeline**
**PROD Pipeline**
Feature Branch
Main Branch
Jira
**Submit**
**Ticket**
**adh_data_science**
Release Branch
**External Data**
**+ Lakehouse Federation**
**Commit**
**PR**
Models are deployed as code through environments to be in line with existing MLOps framework

## Slide 16 — Path to Production - Advanced SQL Analysts

**Pilot Environment**
**(DEV-User)**
Data
**UAT Environment (UAT-User)**
**PROD Environment (PROD-User)**
**DEV Pipeline**
**UAT Pipeline**
**PROD Pipeline**
**External Data**
**+ Lakehouse Federation**
Feature Branch
Main Branch
**adh_data_model**
Release Branch
**PR**
**Submit Ticket**
Jira
**Commit**

## Slide 17 — Path to Production - IT Developers - Advanced

**Pilot Environment**
**(DEV-User)**
Data
**UAT Environment (UAT-User)**
**PROD Environment (PROD-User)**
**DEV Pipeline**
**UAT Pipeline**
**PROD Pipeline**
Feature Branch
Main Branch
**other_git_repo**
Release Branch
**External Data**
**+ Lakehouse Federation**
**adh_general**
Master Branch
Dev Branch
**PR**
**Download**
**PR**
**Submit Ticket**
**DABs**

## Slide 18 — Pathways to Production

All Personas: Develop in Dev user and write to UAT_pilot for testing. Once they are ready for code to be moved to production, they raise a JIRA Ticket. The Tagging Strategy would provide clear observability to understand the costs associated with each Pilot Team.
SQL Analysts: Iterate prototypes to validate business value. Afterwards, they adapt their queries to a standardized template. The completed template is handed over to the DNA team with a list of artefacts.
Speaker notes for each pathway

## Slide 19 — Pathways to Production

Data Scientists: Data scientists leverage MLFlow for tracking whilst developing. Their code is captured in notebooks in a repo that also contains a standardize MLOps DABs template. Their code is pushed to a feature branch of the adh_data_science repo (deployed as code) before making a pull request to the main branch.
IT Developers Standard: Start development in a feature branch with the dbt-sql DABs template or create a customized version. Afterwards, initiate CI/CD with a JIRA ticket and a PR. A DNA team member picks up the bundle and starts productionising through Pipeline workspaces.
Speaker notes for each pathway

## Slide 20 — Pathways to Production

IT Developers Advanced: Allowed to use SDLC practices to develop provided that it conforms to best practice principles of the PfT environment.
We recommend they use Databricks Connect instead of the VS Code extension because it is more robust.
We see no issue with the team running it’s own repo and then creating a feature branch as required in the adh_data_model repo.
Github Actions adds complexity, but ensures that the master branch of merchant_git_repo is always deployed to Databricks. I.e. It removes the risk of a de-sync between Databricks and Git Master.
Speaker notes for each pathway

## Slide 21 — Req 4: User Segmentation - Tailored Access

Databricks Recommended Solution: Pathway to Production + Catalog Redesign
Support three user cohorts:
Advanced Data Scientists: Develop prototype ML models and transition them into production via MLOps.
Business Analysts: Create scratchpads and prototypes without requiring advanced engineering skills.
IT Developers: Build workload features that can later be redeveloped into production assets.
**Our Understanding:**
The developers in the pilot for teams workspace are comprised of the three above profiles.  Any solution should reflect their differing needs.
*Requirement*
Move development work to the Dev User workspace, provision Team Catalogs, and promote to UAT by writing to the team’s UAT Schema.
Enforce the following standards on these Pilot for Teams user types:
Developers: Utilize a feature branch, that incorporates dbt and DABs to promote through environments.
Business Analysts: A handoff checklist that includes: Business Justification, Working SQL Scripts, Source Tables, Expected Outputs, and key stakeholders.
Data Scientists: Parameterized Code using widgets, Utilizing DABs.
*Recommended Solution*
**Pros:**
**Meets users where they are at.**
**Improve user experience (Developer and Business users) and setup the Databricks platform for greater scalability.**
**Workspaces are organized around a common use case, simplifying access, auditing and observability.**
**Considerations:**
**Further upskilling required of Pilot for Teams users.**
**Development teams obliged to conform to DNA standard.**
**No enforcement of Pyspark on developers.**

## Slide 22 — Workspaces: Catalog Binding & Persona Mapping

DEV Pipeline
dev
test
uat
train
prod
pilot*
Development environment
TEST Pipeline
dev
test
uat
train
prod
pilot*
Infrastructure deployment testing
, occasional DE work
UAT Pipeline
dev
test
uat
train
prod
pilot*
User Testing and Verification
TRAIN Pipeline
dev
test
uat
train
prod
pilot*
Pre-prod testing releases
PROD Pipeline
dev
test
uat
train
prod
pilot*
Production workload support
DEV User
(Pilot)
dev*
prod
pilot*
Pilot for Teams development
TEST User
Not actively used unless necessary
UAT User
prod
pilot_uat
Infrastructure testing
, Pilot UAT, UAT
TRAIN User
Not actively used unless necessary
PROD User
prod
Consumption of Prod data, Alerts/dashboards/ML
Admin Access to All Workspaces
uat
Consumption
Only
Consumption
Only
test
prod
train
prod
pilot_uat

## Slide 23 — Requirement 5: Governance - Auditing & Monitoring

Recommended Solution: Tagging
Implement Checks and Balances to prevent runaway growth of pseudo-production assets.
Monitor asset usage, cost, longevity, and quality as indicators of value of inefficiency.
**Our understanding:**
Alinta want users to be able to develop freely in the pilot workspace, however they want a systemic fix to prevent the workspace from becoming bloated over time.
They also want to monitor this workspace for potential pseudo-production patterns,  as well as metrics like usage, cost, quality, etc.
*Requirement*
Implement a tagging strategy that mandates that every data object requires tags that support lifecycle management, cost allocation and governance reporting.
Resources deployed as part of the existing Terraform process will be automatically tagged based upon values set by the DNA team.
User created objects will be updated with tags via an automated tagging job.
This approach scales as required, will not impact any other processes and can be extended as needed.
*Recommended Solution*
**Pros:**
**Observability becomes even more granular.**
**Supports observability for finops, compliance, and lifecycle management.**
**Ease of implementation, thanks to tagging policies and cluster policies.**
**Considerations:**
**Ongoing management of jobs is an additional responsibility for the DNA team.**
**Risk of archiving work that later needs to be restored.**
**Effort required to decide on most appropriate default tag for each situation.**

## Slide 24 — Requirement 6: Automation - Orchestrated, Scalable

Recommended Solution: Four automated processes
Automate resource deployment and governance processes to reduce friction and improve time-to-value.
Provide visibility into asset lifecycle management.
**Our understanding:**
The DNA team already have enough demands upon the time. Any solution should favor automation where possible.
The best candidates for automation are highly repeatable actions like resource deployment and governance.
The solution should report upon asset life cycles at scale.
*Requirement*
**Four separate automation jobs:**
**Tagging job**
- to gain visibility on the whole workspace, tags need to be applied objects created by users outside of the DNA team.
**Security job**
- By default, creators of objects have manage privileges. This job automates the removal of those privileges.
**Monitoring Job**
- A job that writes tag metrics to a materialized view, to support a reporting dashboards.
**Cleaning Job -**
This enforces that 90 day rule and systematically cleans the workspace.
The tagging and automation jobs provide the information needed for granular reporting.
*Recommended Solution*
**Pros:**
**Provides a scalable solution to cleaning and monitoring the PfT workspace.**
**Prevents access creep of data by centralizing governance.**
**Ease of implementation - tags easily accessible by databricks jobs.**
**Considerations:**
**Loss of manage privileges and automated cleaning adds friction for end user.**
**Risk of increased workload for DNA answering request for exceptions.**
**Risk of archiving work that later needs to be restored.**

## Slide 25 — Future State - Automation Jobs Pt. 1

Tagging and Security Flow

## Slide 26 — Future State - Automation Jobs Pt. 2

Monitoring and Cleaning Flow

## Slide 27 — Requirement 7 : Archive or Transition Data

Recommended Solution: Automated Archiving and Transitioning
No strict regulatory mandates exist but ensure that unused or valuable data is either archived or transitioned into production efficiently.
**Our understanding:**
Orphaned data presents a regulatory risk, as not enough is known about it to classify and manage it correctly.
Instead of governing an ever expanded amount of data, Alinta seek a solution where data to either moved to production or archived.
*Requirement*
Leverage tags to apply lifecycle management tags to objects as part of the nightly tagging job.
Any object that has an age of over 90 days and does not have an lifecycle management tag that denotes it should be preserved, that object is archived..
Any object that does match above, falls into one of two categories:
Is currently being moved to production.
Has been granted an exception.
*Recommended Solution*
**Pros:**
**Provides a scalable solution to cleaning and monitoring the PfT workspace.**
**Prevents access creep of data by centralizing governance.**
**Supports observability for finops, compliance, and lifecycle management.**
**Considerations:**
**Loss of manage privileges and automated cleaning adds friction for end user.**
**Risk of increased workload for DNA answering request for exceptions.**
**Risk of archiving work that later needs to be restored.**

## Slide 28 — Requirement 8 : Cost Management and Observability

Databricks Recommended Solution: Tag based observability
Monitor costs associated with pilot environments as indicators of inefficiency or poor design.  Avoid chargebacks but maintain transparency in resource usage.
**Our understanding:**
Increase the granularity of observability so that we can better understand how teams and users are using the platform. Chargeback is not yet required, but we should be able to accurately understand how teams drive cost. Bonus points if the approach can one day be extended to support chargeback.
*Requirement*
Implement a tagging strategy that mandates that every data object requires tags that support lifecycle management, cost allocation and governance reporting.
Build the following reports and incorporate them into your operational processes:
**Finops Dashboard:**
Extend existing finops reporting to leverage the increased granularity provided by tagging.
**Secops Dashboard:**
Report on Access Drift, Shadow IT and anomalous usage patterns.
**Lifecycle Management Observability:**
Report of the state of objects within your workspace, to identify trends in long development cycles, long-lived assets, and shadow IT patterns.
*Recommended Solution*
**Pros:**
**Observability becomes even more granular.**
**Supports observability for finops, compliance, and lifecycle management.**
**Ease of implementation, thanks to tagging policies and cluster policies.**
**Considerations:**
**Additional development required to build the four dashboard.**
**Reviewing the dashboards become an additional responsibility.**
**Effort required to decide on most appropriate default tag for each situation.**

## Slide 29 — Slide 29

## Slide 30 — APPENDIX

## Slide 31 — Key Recommendations

Define a consistent team module (catalog, grants, cluster, tags, etc.) in Terraform.
Implement a new
**Entitlements Framework**
within Unity Catalog to manage data access. This creates a unified, auditable system where every access grant is logged in a central Delta table, complete with business justification and approval workflows.
This embeds governance directly into the data platform, providing real-time visibility and automated compliance monitoring.
Data Accessibility
User Segmentation
Include cluster creation and tags in Terraform team modules.
Create the
operations.entitlements
Delta table to act as the central audit log.
Integrate the framework with the existing ServiceNow ticketing process for managing access requests.
Develop monitoring workflows to identify unusual access patterns and automate compliance reporting.
*Recommendation 1: Scaled Governance Solutions*
*Recommendation*
*Requirements Addressed*
*Next Steps*

## Slide 32 — Key Recommendations

Redesign the Unity Catalog architecture by transitioning from the current single 'Pilot' catalog to a
**domain-based architecture**
. Each team will have a dedicated catalog (e.g.,
pilot_retail, pilot_finance
), which provides better data isolation, simplifies access control, and enhances scalability without requiring changes to existing Entra AD groups
Controlled Collaboration
Formally align on and define the data domains that will map to the new catalogs.
Adjust Terraform scripts to provision new teams with a dedicated catalog instead of a schema.
Plan the migration of existing assets to the new domain catalogs, initially preserving schema names to minimize disruption to user scripts.
*Recommendation 2: Workspace & Catalog Design*
*Recommendation*
*Requirements Addressed*
*Next Steps*

## Slide 33 — Catalog redesign

**Schema-based Domains**
**Catalog-based Domains**
Unity Catalog
Unity Catalog
Catalog: pilot
Catalog: pilot_corpfinance
Schema: pilot_dna
v_fact_journal_lines
fact_jornal_lines
Schema: pilot_insights_analytics
transactions +
Row Filters and Column Masking
Schema:
raw_credit
dim_apa_asset
mv_dim_apa_asset
mv_fact_jornal_lines
v_fact_
committed_spend
fact_committed_spend
Schema: pilot_corpfinance
mv_fact_committed_spend
transactions +
Row Filters and Column Masking
Schema:
structured_credit
transactions
+
Row Filters and Column Masking
Schema:
dm_credit
We recommend switching to domain-separated pilot catalogs as the business scales
Key Points:
Scalability and Operational Simplicity
Self sufficient teams
Allows for a schema per use case / PoC
Governance and Security Boundary
Clear security boundary for each team
Simplified governance
Simplified tagging, cost tracking and reporting
Improved end-user experience
Clear view of all teams data assets
Easier to organise projects / PoC’s
Easier to manage naming collisions
Concerns Raised:
High effort to migrate existing content. Mitigation:
Only content that is considered pre-prod or under extension needs to be migrated. The rest will be removed within the default LCM period.
Migration can be largely automated, to reduce effort.

## Slide 34 — Key Recommendations

Implement a comprehensive and mandatory tagging strategy for all resources, enforced through
**Unity Catalog and automated housekeeping jobs**
.
This strategy defines standard tags for cost allocation, governance, and lifecycle management. Tags will be enforced at creation via cluster policies and applied retroactively via a nightly automation job.
Governance and Monitoring
Cost Management
Regulatory Compliance
Formally agree on the final set of tags to be used for different resources.
Determine the specific tags required for Pipeline vs. Pilot assets.
Update Cluster Policies to enforce tagging rules on compute clusters at creation time.
Develop and deploy the nightly automation job to apply default tags to any untagged, user-created assets.
*Recommendation 3: Tagging and Observability*
*Recommendation*
*Requirements Addressed*
*Next Steps*

## Slide 35 — Future State - Observability

Tagging Supports team based reporting

## Slide 36 — Future State - Observability

Tagging enables you to see how teams are driving usage

## Slide 37 — Key Recommendations

Deploy four key automation jobs using
**Terraform**
and
**Databricks Asset Bundles (DABs)**
to maintain a clean, secure, and observable workspace.
These jobs will handle:
**1. Tagging**
(applying missing tags),
**2. Security**
(revoking creator admin privileges),
**3. Monitoring**
(preparing usage data), and
**4. Cleaning**
(enforcing the 90-day asset lifecycle).
Automation
Regulatory Compliance
Develop the four automation jobs using the Databricks Python SDK and configure them to run as service principals.
Create audit tables for each job to log all actions performed, ensuring full traceability of automated changes.
Schedule the jobs to run on a daily basis using Databricks Workflows to ensure consistent governance.
*Recommendation 4: Automation*
*Recommendation*
*Requirements Addressed*
*Next Steps*

## Slide 38 — Future State - Automation Jobs Pt. 1

Tagging and Security Flow

## Slide 39 — Future State - Automation Jobs Pt. 2

Monitoring and Cleaning Flow

## Slide 40 — Key Recommendations

Establish a standardized production pathway for all pilot work using
**Databricks Asset Bundles (DABs)**
,
**Git-backed Repos**
, and
**automated pipeline promotion**
.
This ensures all assets—from all personas—follow a consistent
Dev → UAT → Prod
workflow, guaranteeing that only validated and versioned assets are deployed to production.
Pathway to Production
Develop standardized DABs templates for different use cases, such as SQL templates for analysts and custom MLOps templates for data scientists.
Upskill business analysts and data scientists to use the new templates and follow the Git-based CI/CD process.
Implement the recommended Git branching strategy (feature → development → main → release) to automate bundle validation and deployment.
*Recommendation 5: Path to Production*
*Recommendation*
*Requirements Addressed*
*Next Steps*

## Slide 41 — Future State - Production Pathway

**DEV Pipeline**
**Prod Pipeline**
**UAT Pipeline**
commit
pull request
deploy as test
merge
release
deploy to prod
check
out
**Dev config:**
Single nodeNo scheduleRun as user…
**Prod config:**
Spark clusterWeekly scheduleRun as service principal…
**Staging config:**
Spark clusterWeekly scheduleRun as user…
All pilot productionisation should follow the structured Pipeline pathway using Databricks Asset Bundles
**IT Developers:**
Leverage
dbt-sql
templates
DBT is run as a task in a job
**DS / ML Engineers:**
Turn the mlops-framework-template into a
custom bundle template
that DS can easily instantiate
**Business Analysts:**
Analysts do not write bundle configuration themselves
Conversion of SQL queries/notebooks to a DBT bundle project

## Slide 42 — Phased Overall Approach

| Weeks 1 & 2 | Weeks 3 & 4 | Weeks 5 & 6 | Weeks 7 & 8 | Weeks 9 & 10 | Weeks 11 & 12 | Weeks 13 & 14 |
|---|---|---|---|---|---|---|
|  |  |  |  |  |  |  |

**Project Management**
**Security : Entitlements Framework**
(Solution and Process Design; Build - App and Enforcement Job; Secops, drift and auditing Dashboard)
**Path to Production**
(DABs templates for Business Analysts. MLOps, Customisation, training, roll-out and adoption)
**Observability**
(FinOps enhancements to existing reports)
**Unity Catalog Migration (optional)**
**Lifecycle Management**
(Finalise Tagging Strategy
Automation: Tagging,LCM Enforcement,Asset Ownership and Security Enforcement. App / LCM Tagging Process, Dashboard)
**Team/User onboarding**
(Process and system integration - (ServiceNow, Entra, CI/CD)
Terraform Build (Modularisation for UC Components, Compute and Security, Budget Policies and Core Tagging)
**RSA Advisory (lifecycle management, team onboarding, UC migration)**
**Databricks activity**
**Alinta/Partner activity**
**Databricks + Alinta activity**

## Slide 43 — Next Steps

| # | Next step | Owner | Date |
|---|---|---|---|
| 1 | Feedback on recommendations and alignment | Alinta | 17-Oct |
| 2 | Internal planning and team alignment | Alinta | 24-Oct |
| 3 | Delivery plan defined after consultation and commitments from Alinta teams | Databricks & Alinta | 31-Oct |
| 4 | Commence implementation | Databricks & Alinta | 10-Nov |

## Slide 44 — Future State - Scaled Governance

Provisioning a Team
Tags
Grants
Security
Compute
Policies
Storage
Pilot for Teams
CorpFinance
CorpFinanceUnityCluster
04-FirbRestricted/Retail/Pilot_CorpFinance
GG-Azure-DataHub-Pilot-CorpFinance
team_name = CorpFinance
Catalog
Schema
Volume
Shared Cluster
Use Privilege on Prod Schema
Cluster Policies
**Workspace Directory: Shared/CorpFinance**
CorpFinanceClusterPolicy
**Pilot Teams by Business Unit**
**Retail**
**CorpFinance		Retail**
**Customer Engagement	RetailCredit**
**DnA			RetailFinance**
**OpAssurance		TPMEast**
**Merchant**
**EnergySupplyTech	TPMWest**
**Corporate**
**RetailAssurance		RiskAnalytics**

## Slide 45 — Catalog redesign

**Schema-based Domains**
**Catalog-based Domains**
Unity Catalog
Unity Catalog
Catalog: pilot
Catalog: pilot_corpfinance
Schema: pilot_dna
v_fact_journal_lines
fact_jornal_lines
Schema: pilot_insights_analytics
transactions +
Row Filters and Column Masking
Schema:
raw_credit
dim_apa_asset
mv_dim_apa_asset
mv_fact_jornal_lines
v_fact_
committed_spend
fact_committed_spend
Schema: pilot_corpfinance
mv_fact_committed_spend
transactions +
Row Filters and Column Masking
Schema:
structured_credit
transactions
+
Row Filters and Column Masking
Schema:
dm_credit
We recommend switching to domain-separated pilot catalogs as the business scales

| Benefits | Reservations |
|---|---|
| Additional level of categorization: can have a schema per pilotCleaner: currently, unspecified schemas mixed in with domain-specific pilots in one catalogBetter physical separation vs taggingClearer feature lineage for MLops | Alinta concern on high effort required to categorize and untangle data - Databricks have observed migrations that have been contained in terms of effort and impact |

## Slide 46 — Current State

Data Science and Machine Learning
DEV/UAT/PROD User
Data Engineering and Stream Processing
DEV/UAT/PROD Pipeline
Batch Ingestion
Domain Based Data Sources*
Stream Ingestion
**Curated Data**
Raw Ingestion and History
Raw
Filtered, Cleaned, Augmented
Structured
Business
Aggregates
Datamart (DM)
Qlik
Enterprise Reporting and BI
Power BI
Lake
SQL and BI
PROD User
Unity Catalog Governance
Access Control
Row Filters Column Masking
System Tables
Cost Controls

## Slide 47 — Future State - Governance Enhancements

Data Science and Machine Learning
DEV/UAT/PROD User
Data Engineering and Stream Processing
DEV/UAT/PROD Pipeline
Batch Ingestion
Domain Based Data Sources*
Stream Ingestion
**Curated Data**
Raw Ingestion and History
Raw
Filtered, Cleaned, Augmented
Structured
Business
Aggregates
Datamart (DM)
Qlik
Enterprise Reporting and BI
Power BI
Lake
SQL and BI
PROD User
Unity Catalog Governance
Tagging
Auditing
**Nightly Automation**
Green boxes identify areas of change in the current state.
Access Control
Row Filters Column Masking
System Tables
Cost Controls

## Slide 48 — Pilot for teams : Requirements → Proposed solution

| What we heard on current challenges | Requirements identified | Identified Gaps | Solution identified to address through below features implementation |
|---|---|---|---|
|  | Data Accessibility | Excessive write access or ad-hoc cross-domain grants.Accidental modification, complex governance | Identity / Access Control-Enforce RBAC. Write only in primary domain. Cross-domain read-only. Temporary roles/service accounts for collaboration. Entitlements Framework to grant access and audit access |
| User Segmentation |  |  |  |
| Controlled Collaboration | Single Pilot catalog with domain-separated schemas only.Risk of cross-domain writes, limited AI/BI scalability | Workspace / Catalog Design-Move toward domain-separated catalogs. Maintain schema separation; define controlled cross-domain access |  |
| Pathway to Production | No standardized promotion or review in pilot workspaces..Ungoverned artifacts, stale Pilot work | Path to Production -Establish Dev → UAT → Prod workflow, DABs, Git versioning; enforce 90-day pilot review |  |
| ML code template requires rewriting of feature table | Path to ProductionSeparate the feature engineering pipeline |  |  |
| Governance and Monitoring | Structured tagging is not currently implemented.Difficult ownership tracking and auditing.If allow users to create tags, be wary of shadow production artifacts persist.Uncontrolled growth-Pilot workspace clutter and orphaned artifacts | Tagging-Define mandatory tagging policies (owner, project, domain, status, archive date). Combine tagging with automated archival/deletion jobs.Automation-Implement archive environment and tagging in a nightly governance job |  |
| Automation | Manual cleanup and promotion.High operational overhead, risk of errors. | Automation-Implement nightly governance jobs |  |
| Regulatory Compliance | No new regulatory compliance requirements have been identified. | As per the current understanding all data within ADH is considered as FIRB, leaning on the safer side. |  |
| Cost Management | Alinta already has a mature report for FinOps observability. However, the report is limited in how granular it can report, as the current tagging approach does not provide sufficient information. | Tagging-By passing the additional tagging information such as ‘team’, the dashboards are able to report on how this tag is trending over time. |  |

Existing architecture predates modern features like Unity Catalog and advanced security protocols, necessitating a redesign
Balancing liberal access with governance to avoid creating a "free-for-all" environment or an overly restrictive one.
Ensuring that non-engineering users (e.g., analysts) are not burdened with learning advanced engineering skills while enabling them to prototype effectively.
Avoiding dependency on non-standardized frameworks that might complicate future enhancements or changes.

## Slide 49 — Future State - Provisioning

## Slide 50 — Production Pathway

There exist cases where a Data Scientist wants to develop with sample datasets (or early iterations/variations of a dataset) that are not yet available in PROD catalog. In these instances, where model development and feature development go hand-in-hand, they may need to develop in DEV User.
In most cases, we recommend that the featurization pathway is separated from model development, so that Data Scientists work only with PROD data.
Alternative DS Journey

## Slide 51 — Slide 51

## Slide 52 — Terraform module of a team

Tags
Grants
Security
Compute
Policies
Storage
{env}
{env}UnityCluster
04-FirbRestricted/Retail/Pilot_{env}
GG-Azure-DataHub-Pilot-{env}
team_name = {env}
Catalog
Schema
Volume
Shared Cluster
Use Privilege on Prod Schema
Cluster Policies
**Workspace Directory: Shared/{env}**
{env}ClusterPolicy

## Slide 53 — Next Steps

Tagging implementation
Align on set of tags
Determine Pipeline vs. Pilot tags
Standardized Pilot Production
Develop SQL standard templates
Develop custom MLops standard templates
Catalog Redesign
Align on data domains

## Slide 54 — Organising the Data Mesh

consume
publish metadata
G
Data Domain
Data Domain
Data Domain
**Data Product**
**Insight**
**Data Product**
**Insight**
**Data Product**
**Insight**
Unity Catalog
G
G
federated governance
G
*Harmonized Distributed Data Mesh*
publish data
Approach
Data domains
will create and publish domain specific data products.
Data discovery
will be enabled by publishing metadata to the common Unity Catalog
Data products
will be consumed in a peer-to-peer way
Domain infrastructure will be harmonized via
platform blueprints, ensuring security and compliance
Self-serve
platform services (domain provisioning automation, data catalog, metadata publishing, …)
Implications
Data Domains need to agree who is responsible for the platform services and owns the admin role for them.
In many cases, a
Central Platform Team
takes over this role. This team might be independent from any data domain. Full automation and self-service will help to avoid this team getting a bottleneck for data product publishing.
Since there is no central infrastructure, every domain needs to develop generic data services (historization, …) themselves
inspired by:  “Harmonized mesh topology”  (https://towardsdatascience.com/data-mesh-topologies-85f4cad14bf2)

## Slide 55 — Organising the Data Mesh

Approach
Data domains
(spokes) will create domain specific high quality data products
Data products
will be published to the data hub
The data hub provides
generic services
platform operations
for data domains.
centralized data publishing
via self service
(data QA is kept in the domains)
Data catalog
via Unity Catalog features
generic data services
like time travel, historization, …
The data hub can also act as a data domain additionally offer own data products
*Hub and Spoke Data Mesh*
G
Data Hub
**Data Product**
Unity Catalog
consume
federated governance
G
publish data and metadata
Spokes
G
Data Domain
Data Product
**Insight**
G
Data Domain
Data Product
**Insight**
Created data product - unpublished
Published data product (per domain)
Implications
Full automation and self-service will help to avoid the data hub getting a bottleneck for data product publishing.
To keep independence, data products in the data hub are partitioned per domain (no integration).
The advantage is that data domains can benefit from centrally developed and deployed data services without own effort.
**Data Product**
**Data Product**
inspired by:  “Governed mesh topology”  (https://towardsdatascience.com/data-mesh-topologies-85f4cad14bf2)

## Slide 56 — Current State Assessment

Pilot space for users to experiment without impacting PROD
Read access to production data and write to dedicated Pilot schema
Use of Databricks Asset Bundles
Structured ETL Pipelines
ML template follows best practices
Workspace and Catalog design

## Slide 57 — Domain-based Catalog

Catalog 1
schema
Tables/Views
Catalog 2
Catalog 3
schema
Tables/Views
schema
Tables/Views
Grant usage to catalog to see schema
Grant usage/create to schema to see tables/views
Grant select/modify to see data, this
**requires usage grants on the schema and catalog**
Owned by central teamUsage Grants performed by central team
Owned by individual teamGrants performed by teams X/Y/ZTeams X, Y or Z cannot share outside of team
Managed by the Metastore owner group

## Slide 58 — Catalog for scalability

| Aspect | Schema-Based Domain Separation (Single Catalog) | Catalog-Based Domain Separation (Multiple Catalogs) |
|---|---|---|
| Terraform Complexity | Moderate. You createschemas within a single catalogusingdatabricks_schemaresources. Access control is managed at the schema level. | Higher upfront. You definemultiple catalogs(databricks_catalog) and their associated schemas. Access control is mostly at the catalog level, simplifying some grants but adding more resources. |
| Access Control / RBAC | Granular, but complex. Must manage schema-level grants carefully to prevent cross-domain write access. Temporary cross-domain access often requires ad-hoc grants. Users can see existence of other schemas. | Cleaner. Users have default roles within their catalog. Cross-catalog access is explicit and easier to reason about. Less chance of accidental write across domains. |
| Governance & Auditability | More complex. Auditing must track schema-level permissions and cross-domain access. Potential for errors if grants are misconfigured. | Simplified. Catalog boundaries align with domains, making auditing, ownership, and lifecycle management clearer. |
| Scaling for AI/BI Workloads | Can work initially, but cross-domain projects require careful schema-level grants and coordination. Risk of accidental writes grows with scale. | Scales naturally. Each domain has its own catalog, reducing cross-domain write risk and simplifying large AI/BI pipelines. |
| Migration / Future-Proofing | Less flexible if you need to split domains later. Changing schema boundaries requires careful refactoring. Architectural consistency w/ Pipeline. | More flexible. Each catalog is naturally isolated. Easy to add new domains or separate teams without reconfiguring existing schemas. |

## Slide 59 — Pilot Catalog strategy

Logical and Physical permission boundaries
Group assignment and grants can be simplified
For MLops, Data Scientists develop feature tables in domain-specific catalog
During “bridging” process, ML Engineer can add metadata to the models about domain-specific catalog to preserve lineage, apply different governance/tests
Domain-based catalog separation

## Slide 60 — Tagging Enforcement - Data Assets

Nightly enforcement job checks if tags are missing, and applies the default values.
DNA adjusts values as requested, either manually or through a second job.
Most user created assets begin without tags - Automated job to enforce tagging

## Slide 61 — Automation

Four* Automation jobs to support clean, observable workspace.

| Job | Function | Schedule | Output |
|---|---|---|---|
| Tagging | Applies any tags not able to be applied at creation | Daily | Tagged objects |
| Security | Enforcement of Access | Daily | Enforced Access to data objects |
| Monitoring | Prepare stats about workspace usage | Dependent on Dashboard refreshes | MVs for Observability tools |
| Cleaning | Enforcement of 90 day rule | Daily | Cleaned EnvironmentAudit Table of actionsPeriodic Alerts to Users |

## Slide 62 — Tagging Design

General tags to apply to all objects

| Tag | Description | Example |
|---|---|---|
| Environment | Identify which environment the resource belongs to. | environment=prod |
| Department | Governance and accountability, group resources by department. | department=technology |
| Team | Grouping one step down from department level | team_name=data_science_and_analytics |
| Sub-team | Not always applicable but a further level of granularity where applicable. | sub_team=BI_ops_support |
| Project / Use Case | Optional tag for DnA to set. Group resource by PoC or use case | use_case=fraud-detection |
| Cost Center | Chargeback / cost allocation | cost_center=CC1234 |
| Availability | Optional, facilitates monitoring and SLA’s | availability=24Hours7Days |
| Team Critical | User settable tag to flag team critical objects | team_value=critical |

## Slide 63 — Tagging Design

Catalog/Schema Level Tags

| Tag | Description | Example |
|---|---|---|
| Environment | Identify which environment the resource belongs to. | environment=prod |
| Department | Governance and accountability, group resources by department. | department=technology |
| Team | Grouping one step down from department level | team_name=data_science_and_analytics |
| Sub-team | Not always applicable but a further level of granularity where applicable. | sub_team=BI_ops_support |
| Project / Use Case | Optional tag for DnA to set. Group resource by PoC or use case | use_case=fraud-detection |
| Cost Center | Chargeback / cost allocation | cost_center=CC1234 |

## Slide 64 — Tagging Design

Table Specific Tags

| Tag | Description | Example |
|---|---|---|
| Compliance / Sensitivity | Data Governance | classification=restricted - Sensitive PI |
| Life Cycle Management State | Tag that is used to enforce Lifecycle Management. Four possible states:Standard (90 day TTL)Pre Prod, is going through productionisation, do not deleteExtension, TTL has been extended, paired with tag belowExemption, this resource is exempt, paired with tag below. | lcm_state=standardlcm_state=pre-prodlcm_state=extensionlcm_state=exemption |
| Extension Period (Days) | Specifies the extension period in days | lcm_extension_period=30 |
| Exemption Type | Specifies the exemption type | lcm_exemption_type=bu_owned |
| Deletion Date | The date which the asset is expected to be deleted.Visible to user | deletion_date=20251122 |

## Slide 65 — Tagging Design

Classic Clusters
SQL Warehouses
Notebooks
Serverless Compute
*Unity Catalog Assets*
*Non-Unity Catalog Assets*
We can infer some missing tags from Unity Catalog objects

## Slide 66 — Tagging Enforcement - Compute

Compute tags can be enforced at time of creation
Classic Jobs*
Classic All Purpose
Serverless Interactive
Serverless Jobs
Vector Search
Model Serving
Cluster Policies
Serverless Policies
DBSQL
DBSQL Serverless
Warehouses

## Slide 67 — Guiding Principles

Existing customer patterns
**Global Software Company**
**Global FSI**
Source:
DAISWT 2024

## Slide 68 — .

## Slide 69 — Powering AI in Energy & Utilities

Key findings from industry & how Alinta is leading the charge
Source:
Economist Impact - Powering AI The outlook in energy & utilities
AI adoption in energy is cautious but growing, driven by complex processes and substantial capital investment.
Top reasons for adopting AI: regulatory compliance, integrating renewables, and market optimization.
Enhanced customer experience is the current leading use case; safety and risk management will be future priorities.
Major impact metrics for AI investment:
Long-term strategic impact (69%)
Scope of AI use (63%)
Risk reduction (61%)
Employee productivity (61%)
GenAI is in early stages—pilots often focus on workflow automation, document processing, and improving developer productivity.
Data governance and security are critical, with cloud data access requiring strict controls.
Only half of surveyed energy sector leaders felt their organization invests sufficiently in AI across both technology and non-tech business functions

## Slide 70 — Powering AI in Energy & Utilities

Key findings from industry & how Alinta is leading the charge
Source:
Economist Impact - Powering AI The outlook in energy & utilities

## Slide 71 — Powering AI in Energy & Utilities

Integrated GenAI into the “Shell-e” platform for contract validation, automated coding, and democratized data access via DIY data tools for employees.
Source:
Economist Impact - Powering AI The outlook in energy & utilities
Led 50 GenAI trial use cases through company-wide brainstorming, enhancing safety guidance, legal document workflows, and developer productivity using GitHub Copilot.
Leveraged AI for optimizing solar panel maintenance and prioritizes secure, catalogued cloud data governance for its renewables businesses.
Customer References

## Slide 72 — Slide 72

**Minutes**
for outage and reliability insights vs days
**Challenge**
Southern Company’s divisions relied on siloed data, inconsistent practices, and slow waterfall processes. During severe weather, outage reporting and crew staging were delayed by days, frustrating customers and straining operations.
**Solution**
They adopted Databricks Data Intelligence Platform with Unity Catalog to unify and govern data, built products like SPEAR (storm prediction) and RAMP (reliability), and shifted to agile two-week sprints with mobile-native, self-service tools.
**Impact**
**Millions**
saved in storm response and staging costs
**100%**
business units using standardized, trusted metrics
©2025 Databricks Inc. — All rights reserved

## Slide 73 — Slide 73

**Challenge**
Alabama Power (APC) needed to modernize its storm management and outage modeling processes. However:
Manual workflows and reliance on spreadsheets delayed forecasting and reduced situational awareness in the field.
Integrating and analyzing massive, high-velocity data streams—from millions of smart meters and multiple sources like GIS, weather, and outage management—was cumbersome.
Ensuring secure, complaint handling of sensitive customer and grid data across diverse datasets posed governance and operational challenges.
©2025 Databricks Inc. — All rights reserved
**Solution**
Used the Databricks Data Intelligence Platform for unified data integration, scalable processing, and real-time analytics.
Built applications (RAMP, SPEAR) for real-time asset monitoring and predictive outage forecasting, optimizing maintenance and storm response.
Centralized data governance, automated reporting, and collaborative tools improved operational efficiency, accuracy, and reliability.
Leveraged AI capabilities and natural language interfaces (Databricks Genie) to accelerate data access, model development, and decision-making
**Impact**
**$17.5M+**
Crew Savings per Year
**>99%**
Reduction in time spent on outage analysis

## Slide 74 — Slide 74

**< 15 min**
Geospatial processing reduced from 9 hours
**Challenge**
Xcel Energy struggled with siloed data, slow manual processes, and underused Databricks. Critical tasks like ignition labeling, geospatial processing, and legacy migration were time-consuming and inefficient, limiting forecasting and risk management.
**Solution**
Partnering with Lovelytics, Xcel Energy modernized Databricks with Medallion architecture, Unity Catalog, and MLOps. GenAI and LLMs automated PII identification, ignition labeling, legacy migration, and rate case analysis, while new data lakes and wildfire models enabled faster forecasting and proactive risk mitigation.
**Impact**
©2025 Databricks Inc. — All rights reserved
**80%**
Legacy Teradata migration accelerated
**20+ Years**
Energy demand forecasting horizon improved

## Slide 75 — Design Permission Structure

Catalog 1
schema
Tables/Views
Catalog 2
Catalog 3
schema
Tables/Views
schema
Tables/Views
Grant usage to catalog to see schema
Grant usage/create to schema to see tables/views
Grant select/modify to see data, this
**requires usage grants on the schema and catalog**
Owned by central teamUsage Grants performed by central team
Owned by individual teamGrants performed by teams X/Y/ZTeams X, Y or Z cannot share outside of team
Managed by the Metastore owner group

## Slide 76 — Data Organization

Use catalogs to segregate your information architecture (by SDLC environment, by BU, or both).
Owners of production catalogs/schemas should be groups, not users.
Only grant USAGE to catalogs/schemas when you want people to see and/or query the contents.
Only allow MODIFY access to service principals in production. Individuals should not update production data manually.
Logical isolation

## Slide 77 — Tagging best practices

*Best Practice Components on Tags*
**What are tags?**
Tags are the best way to track what’s happening across your databricks environment.
They are structured as Key:Value pairings. Eg Business Unit (key): 100,101,102,103 (value)
**What Keys are right for you?**
It’s recommended that everyone apply General Keys, and for organizations that want more granular insights, they should apply select high specificity keys that are right for their organization.
**General:**
(Everyone should start w/ these)
Business Unit
Project
(optional)
Environment
(dev/test/prod)
**High Specificity:**
Builder
(SI partner name, Built on, etc.),
Version, Role, Product, Service, Customer, Other / Custom
**Keys (categories to use)**
Tags can be applied
on clusters, SQL Warehouses, and on Jobs
(Check
account console usage dash for tag coverage data
)
For applying tags to serverless usage leverage Policies (
link to guide
) (Also available: PublicPreview Feature*)
*The PuPr mentioned here only apply for Serverless Notebook/ Jobs, DLT

## Slide 78 — Future State - Production Pathway

Develop SQL transformations with some high-level standards (e.g., one query per transform)
Submit ServiceNow ticket with link to notebooks and queries
*IT Developers*
*Business Analysts*
Develop models using custom DABs bundle template
Submit ServiceNow ticket pointing to project bundle
*Data Scientists*
User Journey matched to User Ability
Develop DBT transformations (silver to gold) in DEV User in a feature branch
Submit a ServiceNow ticket and/or a PR inside IT repo to initiate CI/CD
