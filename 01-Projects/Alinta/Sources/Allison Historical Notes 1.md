---
source: https://docs.google.com/document/d/1gGS1Q41fcjDvoMQe6yz9cl3ak5kVcduM6E_0jWGtdlc/edit
type: google-doc
captured: 2026-03-21
description: Allison Chau's historical notes — Alinta Energy Pilot for Teams Recommendations including workshop feedback, team requirements, governance decisions
---

Alinta Energy Pilot for Teams

**Prepared by Databricks**

Billy O'Connor, Sr. Solutions Consultant

Allison Chau, Resident Solutions Architect

Erik Seefeld, Sr. Resident Solutions Architect

Last updated: September 15, 2025

#

# Overview

Alinta Energy is one of Australia’s largest energy retailers and generators, serving over one million customers across the country with electricity and gas. To accelerate innovation and collaboration, Alinta is adopting Databricks to power a sandbox workspace (**Pilot for Teams**) that will serve as the centralized environment for teams across the organization to conduct data-related work (i.e., data analysis, data exploration, machine learning, etc.).  Over time, this workspace can act as the launchpad for production-grade pipelines, enabling business teams to generate insights and business value more rapidly.

This project is critical for Alinta because the company needs to empower its business units with modern analytics while maintaining trust in data quality and control over costs.

## Project Scope and Goals

As part of this engagement, our architecture design will establish a scalable and governed foundation for Alinta’s Databricks environment, aligning to best practices and innovative standards across data and AI.

Our objective is to provide a solution design for the Pilot Workspace that addresses the following key areas:

**Architecture and Workspaces**

- Review and recommendations on Databricks Workspace design and consumption patterns.

**Automation**

- Automated onboarding and offboarding of teams, resources and data assets

- Automated maintenance routines for the ongoing clean-up of content and assets

**Governance:**

- High-Level / guidance for an entitlements framework to enforce appropriate access levels across all aspects of the Pilot Workspace.

- A tagging strategy to facilitate monitoring, observability and controls at scale.

- Ensure complete visibility and governance of all assets created within the workspace.

**Observability**:

- A design and guidance on enabling the observability of usage patterns, data lineage, and cost attribution across the workspace.

**User / Developer Experience:**

- Assess the current end-user and developer experience.

- Provide actionable recommendations to enhance usability, productivity, and overall experience, where necessary.

**Path to Production:**

- Review the path to production and where necessary, provide guidance on best practices to reduce friction and accelerate delivery.

## Current State

### Architecture and Workspaces

*Fig. 1. Current Workspace and Catalog layout*

Alinta operates multiple Databricks workspaces, including Dev / Test / Prod **Pipeline** workspaces for engineering teams and Dev / Test / Prod **User** workspaces for broader business access. The Prod User workspace, i.e., **Pilot for Teams**, exists to provide relaxed access for business teams. Personas include business analysts, SQL analysts, advanced analysts, and data scientists, with different access needs. Cross-business access is possible but ad hoc.

*Fig. 2. Alinta Data Hub Architecture (Dec 2024)*

Personas:

- Business Persona (need access to Pilot space to check work) - Pattern of creating asset in User Prod, sharing access with collaborators, and not advancing to production.

- SQL Analyst producing outcomes for the business, verified by the business

- Advanced SQL Analyst

- Data Scientists: Data Scientists are very data scientist focused. Do not use Spark, follow any standards, no QA.  (Prod User -> Productionize w/ Team, Development Cycle / Refactor process in Dev Pipeline -> UAT (Validate) -> Prod Pipeline). Currently sits within the IT team.

Should data scientist access to schemas depend on subject matter? Yes. E.g, Retail group, Merchant, Corporate

Prod User workspace <- all usage. No workloads. Create Jobs, Notebooks, interactive clusters. SQL Warehouse sits in that environment used for all business interactions with the data.

Prod User Workspace - Provides production data and production readiness guarantees. (Why would you ever use Dev User? - Only example sounded like a data scientist that needed infrastructure changes?)

Current workspace structures do not fully support a federated path to production, and elevated access levels to various resources (SQL Editor, notebooks, cluster management, dashboards, jobs) are not consistently standardized.

**Areas of Friction:**

- Workspace hierarchy and naming conventions are unclear, making access and permissions inconsistent.

- Cross-business and cross-team collaboration is limited by ad hoc access controls.Lack of a formalized federated production pathway complicates governance and artifact lifecycle.

- Permissions for different roles are not clearly ranked or documented.

“We’ve not really setup a federated path to production” - Andrew

“Is it fair to say that the Workspace structure is up for reassessment.” - Andrew

### Automation

Terraform is used to deploy clusters, manage permissions, and push jobs, integrated with Git CI/CD pipelines. ServicfeNow handles AD group requests manually, and SCIM provisioning is employed for users and group management. DBT is partially used in Gold-layer data assets, but data scientists generally do not leverage it. Automation exists for pipeline deployments, but identity management and lifecycle automation could be further extended.

Terraform: Clusters, permissions for user groups, deploys all jobs into the environment, Git CI/CD to do all the checks, Even into Dev (deploy backwards once you’ve hit UAT) to make sure Dev stays up to date, environments created through Terraform.

Some data assets being created by dbt.

**Areas of Friction:**

- Manual AD group provisioning via ServiceNow introduces delays and potential inconsistencies.

- Terraform modules are fragmented and not fully standardized or reusable.

- DBT adoption is limited among data scientists, restricting workflow standardization.

- Limited automation for identity and access lifecycle management beyond SCIM.

### Governance

Governance is minimal and largely manual. Tagging practices (billing, DevOps, teardown) exist but are inconsistent. Artifact lifecycle management, such as enforcing the 90-day rule for pilot artifacts, is partially implemented but not consistently enforced. Business users have limited mechanisms to share work across units in a controlled, time-bound manner. Unity Catalog policies and access control guidelines are partially in place but not universally applied.

90 Day rule: Solve for pilot teams creating their own data warehouse, getting handful of business team members in there and pointing reports to it without properly productionizing

Business users do not currently have a means to share work with Business units outside of their team. Need a way to enforce granting business users timebound access. Done in theory presently, but timebound nature is not often enforced.

**Areas of Friction:**

- Lack of formalized governance policies and clear ownership of assets.

- Inconsistent application of tagging and lifecycle rules, leading to potential cost and compliance issues.

- Time-bound access enforcement is weak, limiting secure cross-domain collaboration.

- Business value and ROI tracking through tags is not standardized. Business value needs to get tracked. Cost needs to be tracked via tags so that we can advise on ROI

“All I expect for data governance is easy access. There is no data governance standard.” - Gavin

“Who’s object is it, who controls access, and what class of object is it?” - Brad

### Observability

Observability of platform usage and workloads is currently achieved through semi-automated reporting. Users and administrators have limited visibility into cost, job performance, and resource utilization. Notifications to encourage productionization or artifact review are not widely implemented.

There are a few recently-developed Observability solutions:

- Fairly extensive dashboards around cost monitoring. Shows user access, auditing what users have got access to, usage stats (not in good shape) created by EY but going down Genie route instead

- Performance of clusters (classic but not as much around SQL warehouses)

- Platform health

- Need something for the 90 Day rule

**Areas of Friction:**

- Limited automated monitoring and reporting for usage, cost, and performance.

- No consistent mechanism for prompting users to productionize or retire artifacts.

- Metrics for ROI, adoption, and governance compliance are incomplete.

### User / Dev experience

Data scientists primarily use pandas and rarely follow Spark or standardized code practices. SQL analysts and business users have varying levels of access, with elevated permissions inconsistently applied. Users face challenges understanding production requirements, and workflows for collaboration and asset sharing are not fully standardized.

Users primarily work in their personal folders rather than shared folders, making it difficult for teammates to access or build upon their work. Jobs and notebooks are often only visible to the creator, preventing group-level collaboration and transparency.

**Areas of Friction:**

- Inconsistent permission levels and workspace access cause confusion and inefficiency.

- Lack of clear guidance on production requirements for business users and data scientists.

- Limited support for collaborative workflows and self-service data exploration.

- Data scientists do not follow QA or refactoring standards, impacting reproducibility.

### Production pathway

Current production pathways are fragmented, with Github Actions workflows used primarily for engineering workflows. ML pipelines are managed via MLflow, but models can have long runtimes and lack standardized QA or code review processes. Feature engineering and deployment workflows are partially manual. Secret management varies, with some uncertainty about using KeyVault-backed vs. Databricks-backed secrets.

“Certain IT teams can have their own path to production”

Azure Devops

Relatively optimized frameworks for engineering; creating the metadata; requirements aren’t that clear for what they’ve built. Should it be a table or a view?

Secret management

Business users can struggle to articulate the requirements that need to move a data product to production. David sees the requirements as something like “Product owner”, “Domain it originated from”.

#### ML Code Refactoring

*Fig. 2., ML production pathway*

Data scientists develop models primarily in pandas within the sandbox workspace and then transition to production-ready code in the development environment by refactoring to PySpark with the help of software engineers. This **refactoring and handover process is time-intensive** and lacks a fully seamless workflow, with minimal ML-specific testing or validation applied during the transition. Data scientists’ access is appropriately limited to User Prod for experimentation, while engineering teams manage Dev -> Prod Pipeline deployments. This setup ensures governance and control but highlights opportunities to streamline collaboration, testing, and code handover practices.

**Areas of Friction:**

- No standardized production pathway for business users or data science teams.

- Handover is resource and time-intensive, requiring close pair programming in a different workspace

- ML pipelines are inefficient, with limited QA, testing, or refactoring practices.

- Secret management practices are inconsistent, potentially creating security or operational risks.

### First workshop notes

-Users to do UAT, need UAT workspace

-Need same workspace configuration in each tier

-Business UAT occurring all the time (Retail Insights granted access to datamarts)

-Infrastructure testing

Qlik (require test and train at times)

- Categorize wine list prod

- Automated cleanup activity

- Utilize some underutilized workspaces

Primarily concerned with bloat of assets, however, also be a clean up of unused workspace items

20 ATB notebooks, 20 jobs

Rename to be deleted, move to separate Archived folder

Devops initiative: explore backups of user workspace nightly

Governance:

Two roles per schema? Read + Read/Write?

Schema per business unit

User / Dev Experience -> Offer training for users?

Opportunity: Deploy Automatic Identity Management

Tagging design (billing, devops, teardown)

Federated support and maintenance

How to get past 90 day rule: Artifacts need to have a status, Awaiting Review, etc.

Once out of ‘In Dev’ suspend automated deletion. (Need to think carefully about this process to make sure that things moved back to ‘In Dev’ after a review are not accidentally deleted)

Automated reporting would be sufficient instead of automated enforcement.

Perhaps a notification to say ‘you’ve been using for greater x days, consider moving to production’

Move from provisioning storage containers to granting volumes in Unity Catalog.

This slide shows steps specific to how DS promote code, but the spirit of this slide is pretty much to say whether analyst or DS, if they want to retain artifacts past 90 days, they have to

User artifacts in PfT will always be deleted after 90 days.

If they want to retain artifacts, they need to submit a ticket and commit the logic behind their asset, which then progresses to

## MLops best practice and framework

# For second workshop:

## Walkthrough Questions:

- **Can you walk us through your current provisioning process?** (User access to Databricks - Workspaces + Data Assets, IACS)Put a request in Service nowSQL Access (SQL Editor User) Prod workspacePilot for schemas access - (Schema, cluster, workspace folder, volume)Is this all the one AD Group - Different data access for different domains of businessBallpark of data access - 15 for data access.Each pilot for teams environment, there’s two groups. How are you defining that environment? Specific business team (retail insights team, merchant team) Mostly a schema where they can create their own objects. Have write access to that schema. Kinda of prototypes of a production asset.Catalogs are independent of workspace - Separate catalog for each environmentOne catalog per an environment (dev, test, prod), but not one per a workspace)Separately, there is a pilot catalog, which sits in prod.

- **How automated is this process?**Provisioning of users - Fairly manual process, automation project in process.This automation works for PowerBI reports.On provisioning of pilot for teams environments - Done manually, have created a terraform stack for this, but put it on hold for now.Need to adjust based on output of this projectDirect access to flat files via mount points - Recommendation to fix

- **Do you (David) think that automation and integration with ServiceNow is a priority?**David does not consider this part of current scope of this project.

- **Are there any pain points in this process that we’ve missed?**Pain point around provisioning access to business users in the pilot paradigm Time bound access would be great - AndrewIn terms of access to storage  - Folder under analytics containerWIthin ADLS container for analytics - Underneath different business folders, underneath team of pilot groupPotentially not set for each pilot environment

- **Do Pilot teams use DBT?**No, not at all in DBT teams.Andrew - We are planning on training one of counter part teams on DBT, but do not think it will be part of pilot. Seconded by Berza. David: We do not want things in pilot to be treated as sudo production.

- **Can you talk us through the structure of your RBAC design and process?**Most users of structured dataDev, test, UAT, Train, ProdTest and train aren’t really usedUser in Dev user can read Pilot CatalogPilot catalog is not currently bound, should be bound to Prod userCan currently see Pilot from any environment if you use serverless.Do have a lot of prod in non prod environments. Not much use of synthetic dataUAT might need to have Pilot data for user being able to compare across workspacesDev Workspace - If a Data scientist wants to raise a change to a model, UAT User - If user raises something to moveDavid - we can’t give people outside of team access to the pipeline workspace, because there are secret scopes which we do not want them to have.

- **Can we confirm your Unity Catalog design? (share workspace architecture diagram)**Datamart area, definitely by business areaStructured area - by source area David - We don’t do tagging at the moment, but there are things which could be useful. Common schema - Date Dimensions, Weather Data

- **Regarding Data Security and Governance, do you have any requirements outside of the principle of least privilege? Privacy, regulatory, RBAC?** We’ve got FIRB which governs geographical (FIRB - Restricted data cannot be accessed from outside of Australia)Too expensive to classify as Ferv restricted, so everything cannot be accessed from outside of Australia. Andrew - Do have some data with more restrictions - Limited? Internal HR Data (not within Databricks). More sensitive data - Vulnerable customers. Both on Databricks, HR in separate. Don’t want to touch limited.Finance is using pilot environment to mesh Limited. Andrew “Lets not try and solve for that.”Information classification handling standards. E.g. Restricted, confidential, No concept of data owners. Data access controls - LOB scope required for this projcet

- **What do you currently have for Observability?**David - Few different observability solutions which we developed recently.Extensive Dashboards around cost monitoringUser Access - Auditing who has what access to whatUsage Dashbaord - Broken, how much users are querying data, what data are the querying. Who’s looking at what, and how oftenPerformance of clusters - Predominantly classic clustersRecently - Platform Health - Includes metrics around Job execution counts, and failure counts, looks at source systems, connection issues, jobs that running within ADF and count stats.

- **Observability: What do you want to monitor with regards to:**Finops - Pretty well coveredUser Experience - Not a lot of this, Metrics around pilot for teams, observability, what they create, what assets are around, are they being used.Secops - Audit - Presently covered by Furios dashboard

- **Can you talk me through your current CI/CD process? Are there any present pain points with this approach?**

- **Which tools do you use for CI/CD? (Terraform, git, ??)** - Github.

##

## Additional Questions

### Non-Client Questions

- How do we enforce tagging on resources on objects? (Do non unity catalog objects like Notebooks actually need to be culled?)

- Do we need a tag for ‘createdAt’ or will it be preserved? Available in the information schema and via API calls.

- Can we grant time-bound access to Azure AD groups? Yes, see Entra ID P2 question below to confirm that Alinta have the correct access.

- What are we recommending governing Access to assets out of Entra and Unity Catalog? Typically, we would recommend Unity Catalog, however there whole stack seems to flow through Entra.

- What is the best way to enforce tagging so that a user cannot change the auto populated tag? E.g. Terraform assigns a team name, and the user changes it?

### Alinta Questions:

Do you have a Microsoft Entra ID P2 or Microsoft Entra ID Governance license? Required for Entra PIM to provide time bound access requirements - Not presently, will be part of a wider discussion.

How feasible is it for us to work in the office? Feasible - Gavin has requested passes for Wednesday/

Should tagging start in Azure? (Required for Workspace level objects, may not be necessary for Alinta) - Yes, client using AD to drive provisioning and Automation, we should not divert from this.

Should workspace restructuring formalize Dev, UAT, and Prod tiers? Yes, the alternative is to keep informal. Informal space can be handled by the Sandbox. However, questions remain about whether we advise a workspace restructure.

# Stakeholders

“We’ve not really setup a federated path to production” - Andrew

“Is it fair to say that the Workspace structure is up for reassessment.” - Andrew

Goal for is ‘speed to market’

“Brad wants it to be tested and working elsewhere, unless it’s the best.” - Jake

“Present 3 options, but have an opinion about the different options.” - Gavin & Jake

“Pilot for teams was Brad’s baby. Very innovative. I haven’t seen this approach implemented anywhere in APAC”. - Gavin

“Don’t over think things. Keep it simple.” - Gavin

“In our team, we look for almost perfection. The rest of the business is happy to half arse things, and you just need to yell the loudest.” - Gavin

“We do not want things in Pilot for teams to be treated as sudo production.” - David

Beyza’s points on PfT

What current Pilot access gives them:

- Access to prod-user workspace

- Storage - Volume: Under prod/analytics/

- Workspace folder for pilot group

- Access to their own user folder in the workspace

- Interactive cluster dedicated to pilot team

- Pilot schema under pilot catalog (separate from prod catalog)

- Read access to data via their existing access to data in PROD

- Ability to create jobs (only visible to them I believe unless they grant access to the team)

- Ability to create and run notebooks

- Ability to create/delete tables within their own schema

# First Playback Session

Low engagement or support Workspace restructure, regarding anything outside of Pilot for teams

Pushback from David around non-engineers in the Pipelines workspace

	Cannot let non-engineers in Workspace with secrets

	Cannot let business users have the ability to modify code repos, cannot let them have the ability to run things

Retention of ‘Test’ and ‘Train’ workspaces required according to David

Test is used from Testing infrastructure changes before pushing to Dev

Train is used as the ‘pre-prod’ environment (although seems to have less data than prod)

Strong pushback on the catalog strategy, reluctance to re-point any query, didn’t feel that value was clearly articulated. Skeptical of 3 level name space

Need to minimize changes due to change fatigue (explain Default catalog option)

Did not seem to be strong buy-in from fixing MLops problem

Tagging and cleanup Strategy - Strong buy-in and support. Needs to add meat to the bones. Indeterminate answer about cleaning notebooks as well. Beyza has pain point of not knowing which notebook to run when in shared

Automatic Identity Management to be fleshed out further - need to confirm Terraform compatibility

## Erik Feedback

Not surprised by pushback from Workspace design given that Brad wasn’t in the room.

Workspace Restructure needs to be included, asked specifically by Brad to include

Need to progress with existing Catalog strategy limited to the Pilot Workspace

Terraform strategy to automate access via AD Groups, no business users in Prod User should have the ability to manage access to even their own data assets (governed via Access Groups)

Andrew is the loudest voice in the room. Typically most correct

Idea: Recommendations by Stakeholder Matrix

## List of Pain Points

- Workspace Architecture:

  - Workspace hierarchy and naming conventions are unclear, making access and permissions inconsistent.

  - Cross-business and cross-team collaboration is limited by ad hoc access controls.

  - Lack of a formalized federated production pathway complicates governance and artifact lifecycle.

  - Permissions for different roles are not clearly ranked or documented.

  - "What’s the best way to organize all the environments?” Workspaces, personas need organization.

  - Pilot teams creating their own data warehouse, getting handful of business team members in there and pointing reports to it without properly productionizing.

  - Cross-business access needs to be solved for.

  - Cater for a business data scientist if they ever exist (tighter controls).

- Automation:

  - Manual AD group provisioning via ServiceNow introduces delays and potential inconsistencies.

  - Terraform modules are fragmented and not fully standardized or reusable.

  - DBT adoption is limited among data scientists, restricting workflow standardization.

  - Limited automation for identity and access lifecycle management beyond SCIM.

- Governance:

  - Lack of formalized governance policies and clear ownership of assets.

  - Inconsistent application of tagging and lifecycle rules, leading to potential cost and compliance issues.

  - Time-bound access enforcement is weak, limiting secure cross-domain collaboration.

  - Business value and ROI tracking through tags is not standardized.

  - Need policies, share policies, let automation serve policies.

  - Access to new datasets and access currently covered by a very manual ServiceNow process (flagged as potential advisory area).

  - No consistent mechanism for prompting users to productionize or retire artifacts.

  - Access is managed via membership of one of two AD groups (noting that DataBricks access is not currently AD-based)

- Observability:

  - Limited automated monitoring and reporting for usage, cost, and performance.

  - Metrics for ROI, adoption, and governance compliance are incomplete.

  - Need a way to observe how tables are being used.

- User / Developer Experience:

  - Inconsistent permission levels and workspace access cause confusion and inefficiency.

  - Lack of clear guidance on production requirements for business users and data scientists.

  - Limited support for collaborative workflows and self-service data exploration.

  - Data scientists do not follow QA or refactoring standards, impacting reproducibility.

  - Access to folder in a Workspace, but people generally work in their own user folder; when on leave, manager or colleagues come asking for access. Need a way to make sure user folder is for personal work vs. shared team work.

  - Jobs need to be visible for the group, not just the creator.

- Production Pathway:

  - No standardized production pathway for business users or data science teams.

  - Handover is resource and time-intensive, requiring close pair programming in a different workspace.

  - ML pipelines are inefficient, with limited QA, testing, or refactoring practices.

  - Secret management practices are inconsistent, potentially creating security or operational risks.

  - How to handle Data Science flow? Hasn’t existed until now.

## Things that were mentioned to include:

- Way to provision access - Via Terraform (no audit), or Unity Catalog

- Something to address multiple notebook copies

## Useful Things found on their Confluence

Page about Pilot for Teams

Data products are grouped by business domain:

- Retail - (MM, C&I)

- Trading & Portfolio Management

- Power Generation & Development

- Corporate Services (Finance, P&C, HS&E, Legal)

- Technology

### **Productionisation**

Pilot users can submit any code they’ve authored to be redeveloped and productionised into ADH by adding a DnA Request to our Jira board.

Having a process productionised means the pilot user will no longer need to

- Look after running and supporting it

- Maintain and update code based on changes in business conditions

- Look after Acceptable Use conditions for any published objects and/or code

#### **The 90 Day Rule**

Once a pilot data pipeline has been stable for 90 days we consider that the "pilot" has either been successful or that it is no longer needed.

- If it is no longer needed, DnA will request that the user remove it from the environment.

- If it has been successful and there's a need to continue to consume the data, the piloted code will be added to the DnA backlog to be productionised. While this is being done, the user can continue to use the pipeline.

| Resource | Type | Component | What | Role |
|---|---|---|---|---|
| Alinta Servco | Active Directory | GG-Azure-DataHub-Pilot-TeamName | AD group that controls write access | Manage who can write to the environment |
| Alinta Servco | Active Directory | GG-Azure-DataHub-Pilot-TeamName-ReadOnly | AD group that controls read access | Manage who can read from the environment |
| (env)adhaeadls\analytics | Storage Account\Container | /04-FIRBRestricted/LoB/Pilot_TeamName (directory); | Shared folder in analytics zone of data lake | Provide storage for data sets both small and very large |
| (env)-adh-ae-adb-user | DataBricks user workspace | Pilot_TeamName | Shared cluster | Provide an extensible environment to manipulate data in |
|  | DataBricks user workspace | Pilot_TeamName | Shared folder | Provide a way to share code in a given team |
| prod-adh-ae-adb-user | Databricks Schema | pilot_teamname | Schema on Databricks | Provide a database on Databricks to enable the team to create Tables/objects to interact with. |

Security

Pilots security leverages the Data Labs one so all contributing members (ie write access) of a Pilot Environment will need to be Data Lab users.

Pilots are designed for org hierarchy-based teams to collaborate and prototype/pilot work and the Pilot for Teams security structure has been designed to support this.

This is managed via membership of one of two AD groups (noting that DataBricks access is not currently AD-based)

- GG-Azure-DataHub-Pilot-TeamName → Users/people inside the org-hierarchy team (ie no outsiders)

  - Data Lake → Write access to a shared folder in the analytics container

  - Synapse → DML write (ie insert/update/delete) along with select inside pilot schema

  - SQL PaaS → Alter access inside pilot schema

- GG-Azure-DataHub-Pilot-TeamName-ReadOnly → Anyone in the organisation not on the Do-Not-FIRB list and any service accounts held by other teams

  - Synapse → Select access inside pilot schema

Data Labs

Labs people generally get the following

- DataBricks - Account on the production user workspace that allows

  - Start/Stop/Submit on a shared cluster of 2-4 workers

  - Own location/folder to store/manage code from

  - Delta query access over the structured container

- Data Lake

  - Read access to the structured lake (access will be via DataBricks given structured is completely stored in the DataBricks delta format)

  - Write access to personal folder in the analytics container

- Synapse

  - Read access to the data vault

  - Read access to the data marts

### List of Contradictions to solve

- Strong pushback from workspace restructure, however Erik indicated that this would be important to Brad.

- Change fatigue: No appetite for redirecting any queries, or using 3-level namespace, yet the scope of this project.

## Gap Analysis

| Area | Areas of Friction |
|---|---|
| Workspace Architecture | “What’s the best way to organize all the environments?” Workspaces, personas need organization.Pilot teams creating their own data warehouse, getting handful of business team members in there and pointing reports to it without properly productionizing.Tiering doesn’t work.Cross-business access needs to be solved for.Cater for a business data scientist if they ever exist (tighter controls). |
| Automation |  |
| Governance | Need policies, share policies, let automation serve policies.Access to new datasets and access currently covered by a very manual ServiceNow process (flagged as potential advisory area).Lineage / Continuity / Traceability (Where does this data originate?). |
| Observability | Need a way to observe how tables are being used. |
| User / Dev Exp | Access to folder in a Workspace, but people generally work in their own user folder; when on leave, manager or colleagues come asking for access. Need a way to make sure user folder is for personal work vs. shared team work.Jobs need to be visible for the group, not just the creator. |
| Production Pathway | Data science models going backwards to Dev and getting promoted again.How to handle Data Science flow? Hasn’t existed until now. |

# Future State/Best Practice Recommendations

Grouping recommendations alongside relevant scope below

**Architecture and Workspaces**

- Review and recommendations on Databricks Workspace design and consumption patterns.

Parked whilst seeking clarificationWhen it comes to workspace design, there is often a tension between the need for collaboration and the need for isolation. Common patterns include separation based on SDLC, region, business unit

A Lakehouse setup can be split into different workspaces to isolate ETL from consumption and different SDLC environments. Our recommendation is to

- Dev, UAT, Prod, Pilot workspaces

- Logical workspaces within Pilot per Business Unit, within one physical workspace

- Groupings of compute, storage, permissions rolled out automatically via terraform

Azure Workspace Limits

*cross domain

**Automation**

- Automated onboarding and offboarding of teams, resources and data assets

- Automated maintenance routines for the ongoing clean-up of content and assets

Steps in process:

- User requests Access via ServiceNow

- Approved request gets added to config table

- Picked up by Terraform automation job which grants AD Access

| Job | Function | Schedule | Output |
|---|---|---|---|
| Tagging | Applies any tags not able to be applied at creation | Daily | Tagged objects |
| Security | Enforcement of Access | Daily | Cleaned Access to data objects |
| Monitoring | Prepare stats about workspace usage | Dependent on Dashboard refreshes | MVs for Observability tools |
| Cleaning | Enforcement of 90 day rule | Daily | Cleaned Environment |

What should be provisioned with each request?

- 2 AD groups that cover read and write to the workspace

- Access to

**Governance:**

- High-Level / guidance for an entitlements framework to enforce appropriate access levels across all aspects of the Pilot Workspace.

- A tagging strategy to facilitate monitoring, observability and controls at scale.

- Ensure complete visibility and governance of all assets created within the workspace.

Tagging Strategy - What needs to be tagged:

- Team

- Project?

- Lifecycle Status

What type of tags should be used?

Governed tags. Cannot enforcement application at creation time for data objects, should use job instead.

**Observability**:

- A design and guidance on enabling the observability of usage patterns, data lineage, and cost attribution across the workspace.

**User / Developer Experience:**

- Assess the current end-user and developer experience.

- Provide actionable recommendations to enhance usability, productivity, and overall experience, where necessary.

**Path to Production:**

- Review the path to production and where necessary, provide guidance on best practices to reduce friction and accelerate delivery.

Beyza Painpoints run through

- Does not seem sold yet on the advantages of moving to catalog based approach (I don’t understand why that might be useful)

- Prod spans across all workspaces (Pipelines and User)

- Presently, as part of provisioning for all teams there is ADLS container created. Agreement between Billy and Beyza that whilst this was useful for Hive-Metatsore days, it is no longer needed as part of the Unity Catalog For part of recommendations - recommendations to move towards Unity Catalog volumes.

- Jobs: Difficulty in identifying which are prod loads and which are ‘pilot’ workloads (all within the Pilot for Teams workspace). Include as part of MLops. Request for user settable tag, but still having jobs shareable by defaut

- Can we provide ‘Can view’ permissions for all jobs created within a team - Maybe cluster policies for jobs compute + budget policies, then nightly run enforces ‘Can view’ permissions

- Perhaps some documentation about the ‘USE Catalog’ argument to minimize impact for any changes

- Distinguishing tag that can be set by User. Eg. “shared’ so that they can set a job to be available (maybe enforce that it needs to be located in the shared folder)

- Need a way to support users storing their passwords without using plain text. Databricks backed secrets would be a good fit.

entra_group_structure:

  naming_convention: "GG-Azure-DataHub-Pilot-{Domain}-{Team}-{Role}"

  standard_roles:

    team_admin:

      group_name: "GG-Azure-DataHub-Pilot-RetailFinance-CreditRisk-Admin"

      databricks_privileges: ["CATALOG_ADMIN", "CLUSTER_MANAGE", "WORKSPACE_ACCESS"]

    team_contributor:

      group_name: "GG-Azure-DataHub-Pilot-RetailFinance-CreditRisk-Contributor"

      databricks_privileges: ["CATALOG_WRITE", "CLUSTER_RESTART", "WORKSPACE_ACCESS"]

    team_reader:

      group_name: "GG-Azure-DataHub-Pilot-RetailFinance-CreditRisk-Reader"

      databricks_privileges: ["CATALOG_READ", "WORKSPACE_ACCESS"]

scim_provisioning_workflow:

  sync_frequency: "40_minute_intervals"

  enterprise_application: "databricks_pilot_for_teams"

  provisioning_scope:

    entra_groups: "automatic_sync_when_assigned_to_enterprise_app"

    user_attributes: "sync_department_cost_center_manager_info"

    group_memberships: "maintain_real_time_synchronization"

  audit_requirements:

    provisioning_events: "logged_to_audit.scim_events_table"

    group_changes: "tracked_with_business_justification"

    access_reviews: "quarterly_automated_with_manager_approval"

-- Catalog-level grants

GRANT USE_CATALOG, CREATE_SCHEMA, CREATE_TABLE, CREATE_FUNCTION

ON CATALOG pilot_retail_finance

TO `GG-Azure-DataHub-Pilot-RetailFinance-CreditRisk-Contributor`;

-- Schema-level grants

GRANT ALL_PRIVILEGES

ON SCHEMA pilot_retail_finance.creditrisk_workspace

TO `GG-Azure-DataHub-Pilot-RetailFinance-CreditRisk-Contributor`;

-- Volume-level grants

GRANT READ_VOLUME, WRITE_VOLUME

ON VOLUME pilot_retail_finance.creditrisk_workspace.filestore

TO `GG-Azure-DataHub-Pilot-RetailFinance-CreditRisk-Contributor`;

-- Compute grants

GRANT CAN_RESTART

ON CLUSTER creditrisk_shared_cluster

TO `GG-Azure-DataHub-Pilot-RetailFinance-CreditRisk-Contributor`;

-- Warehouse grants

GRANT USE_WAREHOUSE

ON WAREHOUSE prod_shared_warehouse

TO `GG-Azure-DataHub-Pilot-RetailFinance-CreditRisk-Contributor`;

mandatory_tags:

  operational_tags:

    environment: "dev | uat | prod"

    department: "retail | merchant | corporate"

    team: "specific_team_identifier"

    cost_center: "financial_cost_allocation_code"

  optional_context_tags:

    sub_team: "specialized_sub_group_within_team"

    project: "specific_project_or_use_case_identifier"

    data_classification: "public | internal | confidential | restricted"

  governance_tags:

    created_by: "automated_user_identification"

    created_date: "automated_timestamp"

    review_date: "automated_90_day_lifecycle_tracking"

    business_owner: "responsible_business_stakeholder"

team_creation_request:

  business_information:

    - business_unit: "Retail Finance"

    - department: "Credit Risk Management"

    - team_name: "Credit Risk Analytics"

    - business_justification: "Customer credit scoring and risk modeling"

    - expected_team_size: "5-8 analysts and data scientists"

  technical_requirements:

    - data_access_needs: "Customer financial data, transaction history"

    - compute_requirements: "Medium shared cluster + SQL warehouse"

    - compliance_requirements: "FIRB restricted data handling"

    - project_timeline: "Ongoing operational analytics"

  approval_workflow:

    - business_manager: "Department manager approval required"

    - it_security: "Security review for sensitive data access"

    - dna_platform: "Technical feasibility and resource allocation"

semi_automated_workflow:

  step_1_manual_configuration:

    - dna_engineer: "Create team YAML configuration from template"

    - configuration_review: "Validate resource allocation and permissions"

    - version_control: "Commit configuration to infrastructure repository"

  step_2_automated_deployment:

    - ci_cd_trigger: "Manual trigger of deployment pipeline"

    - terraform_execution: "Automated infrastructure provisioning"

    - validation_testing: "Automated verification of team resources"

cost_implications:

  dev_experimentation:

    - "100+ failed experiments per successful model"

    - "Rapid iteration with smaller datasets"

    - "Cost-effective compute for trial-and-error development"

  prod_direct_development:

    - "Full production data processing costs for each experiment"

    - "Production-grade compute costs for experimental work"

    - "Inefficient resource utilization for iterative development"

dev_user_workspace_characteristics:

  data_access:

    - "Dev catalog with development/sampled datasets"

    - "Synthetic or masked data for privacy compliance"

    - "Representative but not live production data"

  resource_management:

    - "Cost-optimized compute for experimentation"

    - "Flexible resource scaling for iterative development"

    - "Development-grade SLAs and availability"

  collaboration_patterns:

    - "Shared experimental environments with other data scientists"

    - "Cross-functional development team access"

    - "Integration with ML engineering development workflows"

# RBAC logic is complex and hard to maintain

entitlements = lookup(group, "entitlements",

  startswith(lower(group.group_name), lower("GG-Azure-DataHub-Pilot")) ?

    local.entitlements_pilot_for_teams_groups :

    startswith(lower(group.group_name), lower("GG-Azure-DataHub${var.env_name}")) ?

      local.entitlements_data_groups :

      local.entitlements_default

)

- Principle of Least Privilege: Many users may have broader access than needed

# Proposed consistent RBAC structure

standard_roles:

  domain_admin:      # Full control within business domain

    privileges: ["ALL_PRIVILEGES"]

  domain_contributor: # Read/write within domain

    privileges: ["SELECT", "MODIFY", "CREATE_TABLE", "USE_CATALOG", "USE_SCHEMA"]

  domain_reader:     # Read-only within domain

    privileges: ["SELECT", "USE_CATALOG", "USE_SCHEMA"]

  cross_domain_analyst: # Approved cross-domain access

    privileges: ["SELECT"] # Read-only across approved domains

# Domain-aligned roles instead of technical roles

business_domain_roles = {

  retail_finance = {

    admin = "GG-Azure-DataHub-Pilot-RetailFinance-Admin"

    analyst = "GG-Azure-DataHub-Pilot-RetailFinance-Analyst"

    reader = "GG-Azure-DataHub-Pilot-RetailFinance-Reader"

  }

  energy_trading = {

    admin = "GG-Azure-DataHub-Pilot-EnergyTrading-Admin"

    trader = "GG-Azure-DataHub-Pilot-EnergyTrading-Trader"

    analyst = "GG-Azure-DataHub-Pilot-EnergyTrading-Analyst"

  }

}

# RBAC based on user attributes

role_assignment_rules = {

  department = "Retail Finance" AND job_level >= "Manager" → retail_finance_admin

  department = "Energy Trading" AND role = "Data Scientist" → energy_trading_analyst

  department = "Corporate Risk" → corporate_risk_reader + approval_required_for_write

}

# Feature promotion workflow

@dataclass

class FeaturePromotion:

    source_catalog: str = "pilot_retail"

    target_catalog: str = "dev"

    feature_table: str = "customer_features"

    validation_rules: List[str] = field(default_factory=list)

    def promote(self):

        # Automated feature promotion with lineage

        self.validate_schema()

        self.copy_with_metadata()

        self.update_lineage_graph()

pilot_retail_finance_catalog = {

  # Only financial DS teams can access

  security_level = "firb_restricted"

  data_classification = "confidential"

}

pilot_merchant_energy_catalog = {

  # Only energy DS teams can access

  security_level = "standard"

  data_classification = "internal"

}

# Retail DS team

retail_features = spark.read.table("pilot_retail.customer_features.daily_metrics")

# Merchant DS team

energy_features = spark.read.table("pilot_merchant.energy_features.demand_patterns")

