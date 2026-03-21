---
source: https://docs.google.com/document/d/1xHKisvagYV7-4s3_tnhAwhiHCPhIBGoVr5cthrnb9mU/edit
type: google-doc
captured: 2026-03-21
description: Alinta Energy Unity Catalog solution design — the primary source document for catalog architecture, security model, and team structure
---

Alinta Energy Pilot for Teams

**Prepared by Databricks**

Billy O'Connor, Sr. Solutions Consultant

Allison Chau, Resident Solutions Architect

Erik Seefeld, Sr. Resident Solutions Architect

Last updated: January 12th 2026

**Document Version:**

| Date | Sections Updated | Version No. |
|---|---|---|
| October 17th, 2025 | v1 | 1 |
| December 2nd, 2025 | 3.2.1, 3.3.1, 3.3.2, 3.5.3, 5Changes marked in red text. | 2 |
| December 2nd, 2025 | 3.3.1Changes marked in red text. | 3 |
| December 4th, 2025 | Added Section 9, marked in red text | 4 |
| December 9th, 2025 | Added Section 5.2, 5.5, Added Arch diagrams and pathways to production diagrams, converted footnotes to own section, minor formatting corrections. Changes marked in green text to distinguish from earlier changes | 5 |
| December 18th, 2025 | Updated diagrams with amendments from presentation, emphasised the Data Scientist pathway is deploying as code. Marked in blue to distinguish from earlier changes. | 6 |
| December 18th, 2025 | Strikethrough and different colors removed. | 7 |
| January 7th, 2026 | Added in minor feedback from Alinta team (spelling and grammar corrections, finer detail clarifications, etc). | 8 |
| January 12th, 2026 | Added section 3.3.6 ‘Controlling Data Sources’ | 9 |

# Executive Summary

Alinta Energy is one of Australia’s largest energy retailers and generators, serving over one million customers across the country with electricity and gas. Within Alinta’s ADH platform, a sandbox workspace (**Pilot for Teams**) exists that serves as the centralized environment for teams across the organization to conduct data-related work (i.e., data analysis, data exploration, machine learning, etc.).  Over time, this workspace is serving as the launchpad for production-grade pipelines, enabling business teams to generate insights and business value more rapidly.

Alinta operates a diverse and mature data ecosystem that reflects its broader business complexity. The organization has built a strong foundation in data engineering and analytics, leveraging Databricks to streamline and scale its data operations. Its ETL pipelines are well-structured, with transformations orchestrated through DBT and a clear progression from raw to refined data across bronze, silver, and gold layers.

As Alinta’s data practice evolves, the team is assessing whether its current design patterns best support future growth, governance, and collaboration. Key challenges include balancing accessibility and control — for example, defining appropriate levels of access for data scientists and ensuring accountability for the artifacts they create.

To address these challenges, Alinta is introducing structured processes such as time-bound “pilot” workspaces. These environments allow users to explore and prototype using production-level data while maintaining governance through policies like a 90-day project review rule. This approach encourages innovation and autonomy without compromising operational discipline.

Looking ahead, enhanced tagging, artifact management, and standardized project templates will help Alinta sustain efficiency and collaboration as its data landscape scales. While the current environment benefits from openness and agility, proactive steps toward stronger governance and domain-based access patterns will position Alinta to meet future security and compliance requirements with confidence.

# Discovery and Requirements

As part of this engagement, our architecture design will establish a scalable and governed foundation for Alinta’s Databricks environment, aligning to best practices and innovative standards across data and AI.

Our objective is to provide a solution design for the Pilot Workspace that addresses the following key areas:

**Architecture and Workspaces**

- Review and recommendations on Databricks Workspace design and consumption patterns.

**Automation**

- Automated onboarding and offboarding of teams, resources and data assets

- Automated maintenance routines for the ongoing clean-up of content and assets

**Governance:**

- High-Level guidance for an entitlements framework to enforce appropriate access levels across all aspects of the Pilot Workspace.

- A tagging strategy to facilitate monitoring, observability and controls at scale.

- Ensure complete visibility and governance of all assets created within the workspace.

**Observability**:

- A design and guidance on enabling the observability of usage patterns, data lineage, and cost attribution across the workspace.

**User / Developer Experience:**

- Assess the current end-user and developer experience.

- Provide actionable recommendations to enhance usability, productivity, and overall experience, where necessary.

**Path to Production:**

- Review the path to production and where necessary, provide guidance on best practices to reduce friction and accelerate delivery.

ADH has been established over the last 6 years, with a solid base being maintained by the DNA team. As they look to open the environment to other teams, we have collected the following additional pain points which will need to be addressed to retain the current quality and reliability of the ADH platform:

- Workspace Architecture:

  - Cross-business and cross-team collaboration is limited by ad hoc access controls.1

  - Some workspaces (Train, Test) are not used, and other workspaces have uncertainty around access patterns (e.g., Should Dev User be for Data Scientists? Should models be promoted through User workspaces for MLflow experiment visibility?) 2

- Automation:

  - Manual Entra group provisioning via ServiceNow introduces delays and potential inconsistencies which could also impact security. Limited automation for identity and access lifecycle management beyond SCIM provisioning. 3

  - Terraform scripts are implemented as modularized stacks and deployable by environment, but team components can be better defined and standardized. 4

- Governance:

  - Lack of formalized governance policies and clear ownership of assets. Users often retain manage permissions within Databricks for any data assets they created, allowing them to bypass the DNA when sharing those assets. Users store shared assets in personal workspace folders, causing additional effort for the DNA team to share those assets when an owner is unavailable. Team jobs are often visible to just the creator, despite the whole team requiring access. 5

  - Inconsistent application of tagging and lifecycle rules, leading to potential cost and compliance issues. Business value and ROI tracking through tags is not standardized, reducing Alinta’s ability to track spend and chargeback. 6

  - Time-bound access enforcement is weak, limiting secure cross-domain collaboration and contributing to workspace bloat. No consistent mechanism for prompting users to productionize or retire artifacts. 7

- Observability:

  - Limited automated monitoring and reporting for usage, cost, and performance. Metrics for ROI, adoption, and governance compliance are incomplete. 8

- User / Developer Experience:

  - Permissions for different roles are not clearly ranked or documented. Inconsistent permission levels and workspace access cause confusion and inefficiency. 9

  - The existing ML template solution is not broadly used. Business analysts also do not use standard templates for their projects. 10

- Production Pathway:

  - No standardized production pathway for business users or data science teams. Pattern for IT Developers (using DBT) is unclear. 11

  - DBT adoption is not used by business analysts, restricting workflow standardization.12

  - ML handover is resource and time-intensive, requiring close pair programming in a different workspace. May need to promote through User workspaces (instead of Pipeline), so that Data Scientists can see MLflow experiments as the model gets promoted. However, models should not be written into UAT or Prod from the User workspaces. Data Scientists want separate workspaces so they can better isolate MLflow experiments from their teams. 13

  - Secrets management practices are inconsistent, creating security or operational risks. 14

The current state architecture is captured in the below diagram:

Figure 1: Current State Architecture

# Solution Components

In this section, we detail the components that make up the final design of an uplifted Pilot for Teams environment. This includes personas of Pilot for Teams users, workspace design, catalog design, tagging strategy, and identity design. The ensuing sections of the document cover the flows of how these components work together to create a well-governed and collaborative environment, meeting Alinta’s business needs.

The solution design recommended by Databricks is captured below:

			Figure 2: Future Architecture if all recommendations are adopted

## Personas discovered

Personas include business analysts, SQL analysts, advanced analysts, and data scientists, with different access needs. Cross-business access is possible but ad hoc.

| Persona | Description | Proposed Tool Set |
|---|---|---|
| Data Scientists | Develop prototype ML models and transition them into production via MLOps. | NotebooksJobsDABsMLflowModel Serving |
| Business Analysts | Create scratchpads and prototypes without requiring advanced engineering skills. | NotebooksJobsDBSQLLakeflow DesignerDABs |
| IT Developers | Build workload features that can later be redeveloped into production assets. | NotebooksJobsDBSQLMV’sLakeflow DesignerDABsApps |

##

## Workspace Design

Alinta operates multiple Databricks workspaces, including Dev / Test / UAT / Train / Prod **Pipeline** workspaces for engineering teams, and Dev / Test / UAT / Train / Prod **User** workspaces for broader business access. Test and Train workspaces are dedicated to infrastructure and pre-production release testing for the Pipeline environment. User workspaces may also be used, when required, to provide isolated environments for Digital teams and to support DevOps infrastructure testing. The Prod User workspace, i.e., **Pilot for Teams**, currently exists to enable access for individual consumers as well as for business teams. The purpose of a “pilot” is to allow business analysts to self-author analytics/reports and pilot their own processes on ADH. It should be designed to have a clear path to the Pipeline Production workspace so that proven pilots can be productionized.

### Option A: Pilot for Teams (Dev/UAT/Prod Separation) (Recommended)

The User workspaces within ADH currently exist to enable user access to ADH (any user outside of DnA developers and vendors). This separation is designed to restrict Pipeline workspaces to data engineering/ML development by DnA, versus data consumption/exploration/pilot work by business and other IT teams.

The proposal is to move all Pilot development to the **Dev User** workspace to ensure the **Prod User** workspace (a production environment for consumption of data) remains clean, stable, and isolated, reducing the risk of pollution and enabling easier support.

Pilot teams will develop pilot assets in **Dev User** while having controlled access to **PROD**, **uat**, **pilot_uat**, and **pilot_<team>** catalogs.

Catalog access justification:

- **prod catalog** will be the primary data source for building pilot work.

- **uat catalog** will be available only when pilot work must align with BAU work by DnA (e.g., ingestion of new SNOW datasets while simultaneously building a pilot on top of that dataset — a rare occurrence).

- **pilot_<team> catalogs** will exist per team, enabling development of multiple logical workstreams via separate schemas.

- **pilot_uat catalog**, with a schema per team, will be used for UAT testing of assets that are near-production-ready but still require business validation.

- Business users will be granted temporary access to their team’s schema under **pilot_uat** for UAT.

- Once UAT is passed, **access will be revoked and objects will be removed** from this schema, followed by a formal productionisation request via the Front Door process.

Business Analysts performing UAT will use the **UAT User** workspace with access to:

- uat catalog (for BAU DnA-managed objects)

- pilot assets under pilot_uat

- prod catalog (for comparing production data to enhancements being validated in UAT)

This explicit workspace and catalog separation also ensures clear** cost attribution**, allowing Alinta to distinguish Pilot Development, Pilot UAT, and Production consumption costs.

Overall, Option A strengthens **environment hygiene, provides clearer lifecycle progression** (Dev → UAT → Prod), **reduces operational overhead** for DnA, and aligns with Databricks best-practice for **controlled multi-environment** user access.

### Option B: Pilot for Teams (SIT/Prod)

Pilot for Teams consists of the UAT User and Prod User workspaces. The end user (e.g. business analyst) gets *read* access to Prod Catalog data, and *write* access into the Pilot catalog. The physical workspace separation of ETL processes (Pipeline) and consumers of the data (User) is best practice, which is already currently implemented in Alinta.

These workspaces maintain strict access controls and are optimized for automated, scheduled data operations that form the foundation of Alinta's data platform. User Workspaces (Dev/UAT/Prod) enable business analysts, data scientists, and domain experts to access curated data for analytics, reporting, and experimentation. This separation ensures that business users can perform self-service analytics without interfering with production data pipelines, while maintaining appropriate governance and security boundaries. This workspace separation pattern represents Databricks best practice, providing clear operational boundaries between data production and consumption while enabling appropriate collaboration and data sharing across business functions.

All personas (Business Analysts, Data Scientists, IT Developers) should develop work in Prod User with write access to Pilot catalogs. Data Scientists and IT Developers can be given access to the Dev User and UAT User workspaces for QA purposes only, with read-only access to Dev, UAT, and Prod catalogs.

### Option C: Federated Teams and Workspaces (Data Mesh)

Databricks does not recommend a Data Mesh as a solution to the pain points encountered by Alinta. A Data Mesh would necessitate major changes to organisation structure at Alinta, and the hiring of new technology personnel to embed into these federated teams. During our consultation with the Alinta team, it quickly emerged that these changes would not be feasible, and so this option was disqualified as a potential solution.

## Data Asset Structure Design

This section outlines the proposed design for Alinta's data asset structure within Databricks, specifically focusing on how assets are organized within Unity Catalog. It evaluates three primary options: Catalog per Team ( for its scalability and enhanced categorization), Catalog per a team whilst moving the Pilot for teams structure into the Dev User Workspace, and a Single Teams Catalog (the current approach, which may become cumbersome over time). Additionally, it addresses Unity Catalog Volumes, recommending the use of Managed Volumes over legacy ADLS container provisioning for improved user experience and reduced overhead. Finally, we recommend Federated Queries as a means to connect to other Database Management Systems. The aim is to establish a governed and efficient framework for managing data assets.

### Option A: Catalog per Team & Schema per Team under pilot_uat Catalog (Recommended – Based on Option A 3.2.1)

This approach introduces a structured, scalable environment and catalog design for Pilot development that aligns with the updated **Workspace Option A** model. It provides a clean separation between **Pilot Development**, **UAT**, and **Production Consumption** while simplifying governance, access, and cost attribution.

**Catalog Design**

Each business team is allocated a dedicated **domain-based catalog** for Pilot development:

- pilot_retail_finance

- pilot_energy_supply_tech

- pilot_credit_risk

- pilot_corporate_services

These catalogs are used exclusively in the **Dev User workspace** and allow teams to create multiple schemas to logically separate ongoing initiatives. For example:

- pilot_retail_finance.forecasting

- pilot_retail_finance.billing_improvements

This aligns to Databricks’ best practice of three-level namespaces (catalog.schema.table) and simplifies policy enforcement and lifecycle management.

**UAT Design**

A dedicated pilot_uat catalog is introduced to support structured user acceptance testing.

Each team is provisioned a **named schema** inside the pilot_uat catalog:

- pilot_uat.retail_finance

- pilot_uat.energy_supply_tech

- pilot_uat.credit_risk

When a team deems a pilot asset ready for business testing, they **deploy it (write/copy tables) from their Dev workspace** into the appropriate schema under pilot_uat. This action is considered a soft promotion of the pilot for business validation.

Business Analysts performing UAT will do so in the **UAT User workspace**, where they have:

- **Read-only access** to their team’s schema under pilot_uat

- **Read-only access** to the prod catalog (for comparison)

- **Read-only access** to uat catalog (UAT of non-pilot assets)

Once UAT is completed:

- Access to the pilot_uat schema is revoked

- The object is removed from pilot_uat

- A productionisation request is submitted to DnA via the standard front-door process

This model preserves clean boundaries while streamlining the promotion process.

**Workspace Access Pattern:**

| Workspace | Purpose | Write Access | Read Access |
|---|---|---|---|
| Dev User | Pilot development | pilot_<team>, pilot_uat.<team> | prod, uat, pilot_<team> |
| UAT User | Business UAT | None | prod, uat, pilot_uat.<team> |
| Prod User | Production data consumption | None | prod only |

**Implementation and Effort**

The main effort will involve recategorizing existing data and aligning assets to their respective team domains, ideally through a phased migration starting with the most bloated or complex areas.

Firstly, this recommendation does not require any changes to AD groups. The existing AD group structure would still be utilized.

# Existing AD groups continue to function

- group_name: GG-Azure-DataHub-Pilot-RetailCredit-DEV

  new_catalog_access: pilot_retail_finance

  existing_permissions: preserved

- group_name: GG-Azure-DataHub-Pilot-EnergySupplyTech-DEV

  new_catalog_access: pilot_merchant_energy

  existing_permissions: preserved

Secondly, the Terraform process would remain largely unchanged only two adjustments required. One adjustment would be to provision groups with a Catalog instead of a schema. Terraform adjustments focus on catalog provisioning rather than group management, simplifying operational overhead while maintaining security boundaries.

# Current approach: Complex schema-level permissions

resource "databricks_grants" "pilot_schema_grants" {

  for_each = local.business_unit_schemas

  schema   = each.value.schema_name

  # Complex conditional logic for mixed security requirements

  dynamic "grant" {

    for_each = each.value.security_level == "high" ? local.restricted_grants : local.standard_grants

    content {

      principal  = grant.value.principal

      privileges = grant.value.privileges

    }

  }

}

# Proposed approach: Simplified catalog-level permissions

module "domain_catalogs" {

  source   = "../modules/pilot-domain-catalog"

  for_each = local.pilot_domains

  domain_name     = each.key

  security_level  = each.value.security_level

  business_units  = each.value.business_units

  # Consistent permission patterns per domain

  admin_group     = "GG-Azure-DataHub-Pilot-${title(each.key)}-Admin"

  contributor_group = "GG-Azure-DataHub-Pilot-${title(each.key)}-Contributor"

  reader_group    = "GG-Azure-DataHub-Pilot-${title(each.key)}-Reader"

}

An additional **pilot_uat** catalog will be created, with schemas provisioned per team (**pilot_uat.<team>**). Ownership of each schema will sit with the respective pilot team, enabling them to directly manage access to their UAT objects as required. This simplifies and accelerates the UAT process by removing the need to raise SNOW requests and wait for access to be granted, which is difficult to manage effectively for a pilot.

If this approach proves difficult to manage in the future, it can easily be switched to the standard access pattern of raising a SNOW request to be added to an AD group (with new pilot-UAT AD groups created per team) and incorporating **pilot_uat** into the Terraform schema grant scripts. As an alternative, the security job (automation) may also be extended to cover this catalog if required.

When moving existing data from within the Pilot catalog, we could minimize any disruption by preserving the schema name. We recommend to begin with by creating a new schema within the new catalog with the same name as the old schema. For example, ‘pilot.pilot_retailcredit.atb_history’ would become ‘pilot_retailcredit.pilot_retailcredit.atb_history’.

Any scripts that are currently using the 3 level name space could adjust with a simple *find and replace*. Any scripts currently using a 2 level name space could be retired by prefixing the scripts with a ‘USE CATALOG’ code block.

This schema will gradually be de-populated as users create new schemas for use, and the 90 day rule enforcement removes any existing tables within this schema.

### Option B: Catalog per Team

This involves transitioning to a domain-based (i.e., team-based) catalog architecture where each business team operates within dedicated catalogs (e.g., pilot_retail_finance, pilot_energy_supply_tech, pilot_corporate_risk). This architectural evolution provides enhanced data isolation, simplified governance, and improved scalability while aligning technical boundaries with business domain ownership.

A domain-based catalog separation is recommended as a best practice for scalability and future-proofing. While not every schema currently shows signs of bloat, several domains, such as Retail Finance (251 tables), Retail Credit (161), Retail Assurance (197), Energy Supply Tech (210 tables and 29 models), would benefit from an additional layer of categorization. In the future state, users can create a schema to house their pilot project. If the project involves cross-collaboration across teams, the cross-collaborator can be granted access to the specific schema. This approach also supports MLOps maturity by enforcing consistent use of a three-level namespace (catalog.schema.table).

Separating teams into domain-based catalogs provides a clear governance and security boundary, while tagging can achieve some level of control, catalog-level isolation offers stronger physical separation of data and simplifies access management. Cross-collaboration can still be governed effectively by granting collaborators access only to a specific schema within a catalog.

**Implementation and Effort**

The main effort will involve recategorizing existing data and aligning assets to their respective team domains, ideally through a phased migration starting with the most bloated or complex areas.

Firstly, this recommendation does not require any changes to AD groups. The existing AD group structure would still be utilized.

# Existing AD groups continue to function

- group_name: GG-Azure-DataHub-Pilot-RetailCredit-DEV

  new_catalog_access: pilot_retail_finance

  existing_permissions: preserved

- group_name: GG-Azure-DataHub-Pilot-EnergySupplyTech-DEV

  new_catalog_access: pilot_merchant_energy

  existing_permissions: preserved

Secondly, the Terraform process would remain largely unchanged. The only adjustment would be to provision groups with a Catalog instead of a schema. Terraform adjustments focus on catalog provisioning rather than group management, simplifying operational overhead while maintaining security boundaries.

# Current approach: Complex schema-level permissions

resource "databricks_grants" "pilot_schema_grants" {

  for_each = local.business_unit_schemas

  schema   = each.value.schema_name

  # Complex conditional logic for mixed security requirements

  dynamic "grant" {

    for_each = each.value.security_level == "high" ? local.restricted_grants : local.standard_grants

    content {

      principal  = grant.value.principal

      privileges = grant.value.privileges

    }

  }

}

# Proposed approach: Simplified catalog-level permissions

module "domain_catalogs" {

  source   = "../modules/pilot-domain-catalog"

  for_each = local.pilot_domains

  domain_name     = each.key

  security_level  = each.value.security_level

  business_units  = each.value.business_units

  # Consistent permission patterns per domain

  admin_group     = "GG-Azure-DataHub-Pilot-${title(each.key)}-Admin"

  contributor_group = "GG-Azure-DataHub-Pilot-${title(each.key)}-Contributor"

  reader_group    = "GG-Azure-DataHub-Pilot-${title(each.key)}-Reader"

}

Thirdly, when moving existing data from within the Pilot catalog, we could minimize any disruption by preserving the schema name. We recommend to begin with by creating a new schema within the new catalog with the same name as the old schema. For example, ‘pilot.pilot_retailcredit.atb_history’ would become ‘pilot_retailcredit.pilot_retailcredit.atb_history’.

Any scripts that are currently using the 3 level name space could adjust with a simple *find and replace*. Any scripts currently using a 2 level name space could be retired by prefixing the scripts with a ‘USE CATALOG’ code block.

This schema will gradually be de-populated as users create new schemas for use, and the 90 day rule enforcement removes any existing tables within this schema.

### Option C: Single Teams Catalog

The current architecture used by Alinta segregates different teams by schema within a single ‘Pilot’ catalog. This is presently fit for purpose, and works to effectively separate data into separate logical containers. Crucially, no additional effort is required to set up this approach as it is already implemented.

However, we do anticipate problems over time with this approach. Primarily, this approach does not allow the team to categorize any of their tables. Over time this approach causes table management to be cumbersome, and forces management to be applied at the level of the individual table.

### Unity Catalog Volumes

Presently, Alinta provisions an ADLS container and corresponding UC External Volume for new teams being onboarded to Databricks. After investigating this process, it appears to be unnecessary and a legacy process from when new teams were onboarded to the Hive Metastore Catalog.

For new teams, we would instead recommend that as part of onboarding they are upskilled in using Databricks Volumes. This prevents the user from needing to switch between Databricks and the Azure portal, as well as abstracting away the file management from the user.

For Databricks volumes, there are two types of volumes available, Managed Volumes and External Volumes. Typically, Unity Catalog Managed Volumes are sufficient for most business users. External Volumes provide an additional advantage that the underlying storage can be accessed easily from outside Databricks, e.g. by Azure Data Factory or Azure Storage Explorer.

Databricks would recommend that Alinta adjust the process to utilize Managed Volumes and to no longer create Storage Account Containers for users. This is because the user requirements for users in Pilot for Teams do not require access to underlying storage, and managing this access brings an unnecessary overhead, especially since external tools can directly access files using URIs, bypassing Unity Catalog.

To prevent the proliferation of volumes within a schema, the creation of a single volume could be part of the team provisioning process, and team members would have their permissions limited to ‘READ VOLUME’ and ‘WRITE VOLUME’. They would not have ‘CREATE VOLUME’ permissions, which would prevent them from creating additional volumes.

### Federated Queries

Query federation allows Databricks to execute queries against data served by other Databricks metastores as well as many third-party database management systems (DBMS) such as PostgreSQL, and MySQL.

When creating foreign connections, credentials are often required. For secure storage of credentials with Databricks, please see section 3.5.4

### Controlling Data Sources

To ensure governance and prevent teams in the Dev User workspace from onboarding new data sources without proper oversight, Unity Catalog permissions should be utilized to restrict teams to only existing data sources. This involves controlling permissions such as CREATE EXTERNAL LOCATION, CREATE STORAGE CREDENTIAL, CREATE MANAGED STORAGE, EXTERNAL USE LOCATION, and CREATE EXTERNAL TABLE.

Additionally, network policies can be implemented to restrict the use of APIs within notebooks. Where necessary, specific web addresses can be whitelisted to allow exceptions.

## Tagging Strategy

Databricks recommends tagging resources for several key reasons, primarily to enable enhanced observability and reporting on business-level metrics. Tagging supports the following initiatives:

- **Cost Allocation and FinOps:** Tags allow organisations to accurately attribute Databricks costs to specific teams, projects, departments, or cost centres. This granular visibility supports FinOps practices, enabling better budget management, cost optimisation, and chargeback mechanisms.

- **Governance and Accountability:** By tagging resources with information like team, project, or owner, organisations can enforce governance policies and establish clear accountability for resource usage and data assets. This helps in understanding who is using what and for what purpose.

- **Security Operations (SecOps) and Compliance:** Tags can be used to classify data by sensitivity (e.g., PII, confidential) or compliance requirements (e.g., FIRB, APPs, GDPR). This will aid in implementing attribute-based access control (ABAC) in the future and ensure that security policies are applied consistently across relevant resources, simplifying compliance audits and risk management.

- **Lifecycle Management (LCM):** Tags, such as lcm_state and creation_date, support managing the lifecycle of data and compute resources. They enable automated processes to identify stale data, enforce retention policies, and delete resources that are no longer needed, preventing data sprawl and reducing storage costs.

- **Operational Visibility and Troubleshooting:** Tags provide context for operational monitoring. When issues arise, tags can quickly help identify the affected team, project, or application, accelerating troubleshooting and reducing downtime.

- **Resource Optimisation:** By understanding how different tagged resources are being utilised, teams can identify underused assets or inefficient configurations, leading to better resource allocation and performance optimisation.

- **Automation:** Tags can be leveraged in automation scripts and CI/CD pipelines to dynamically provision, configure, and manage resources based on predefined tagging rules, increasing efficiency and reducing manual errors.

### General Tags

By default, we recommend having a default value for each tag. This tag will be applied by the nightly enforcement job to any object which is missing a tag.

However, for objects which sit below a tagged object within Unity Catalog, we can sometimes infer what their value should be from the hierarchy within Unity Catalog. For example, if a schema is tagged to a business unit, and the table within the schema is missing a business unit tag, then we can infer the business unit tag for the table from the schema.

Databricks recommends the following general tags to be applied to all assets in the Pilot workspace.

| Tag | Description | Example |
|---|---|---|
| Environment | Identify which environment the resource belongs to. | environment=prod |
| Department | Governance and accountability, group resources by department. Existing tag ‘ADH_BusinessUnit’ might serve here. | department=technology |
| Team | Grouping one step down from department level | team_name=data_science_and_analytics |
| Sub-team | Not always applicable but a further level of granularity where applicable. | sub_team=BI_ops_support |
| Project / Use Case | Optional tag for DNA to set. Group resource by project or use case | use_case=fraud-detection |
| Cost Center | Chargeback / cost allocation | cost_center=CC1234 |
| Availability | Optional, facilitates monitoring and SLA’s | availability=24Hours7Days |
| Team Value | User settable tag to flag team shared objects | team_value=shared |

The team value tag is a user settable tag to flag objects such as jobs or tables that are valuable to their team. For example, a job which outputs a table that the whole team depends on. This would allow teams to flag jobs or data assets that are especially important to their team, and thereby reduce the support time required from the DNA team.

### Classic Compute Tags - Cluster Policies

When creating classic clusters, Databricks recommends using Cluster Policies to ensure that clusters comply with standards set by the DNA team. There is also the option to prevent users from creating classic clusters and to instead have clusters created for them (either through Terraform or by a workspace Admin), however this would restrict the ability of users to run jobs, and increase the demands placed upon the DNA. As such, our best practice recommendation is to restrict the configuration of clusters that can be created, instead of removing the ability for users to create clusters.

### Serverless Compute Budget Policies

Serverless Compute Budget policies automatically apply tags to compute for users using serverless notebooks or serverless warehouses. This allows queries, and therefore cost, to be assigned to a particular user.

A budget policy will be required for each team, and each user will need to be assigned to their team's serverless budget policy.

Presently, serverless budget policies are not generally available.

### Unity Catalog Tags

Databricks recommends that the 90 day policy enforcement rule is only applied to tables and not to catalogs and schemas. This is because Catalog and schemas will continue to be deployed as part of Alinta’s pilot team provisioning process.

Tables will be created by users, and we do not recommend allowing users to set their own tags as this may interfere with governance standards. Additionally, enforcing tags to be set at the time of creation is not yet supported. Instead, tag compliance will be enforced via an automated nightly job.

**Catalog / Schema Tags**

| Tag | Description | Example |
|---|---|---|
| Environment | Identify which environment the resource belongs to. | environment=prod |
| Department | Governance and accountability, group resources by department. | department=technology |
| Team | Grouping one step down from department level | team_name=data_science_and_analytics |
| Sub-team | Not always applicable but a further level of granularity where applicable. | sub_team=BI_ops_support |
| Project / Use Case | Optional tag for DNA to set. Group resource by project or use case | use_case=fraud-detection |
| Cost Center | Chargeback / cost allocation | cost_center=CC1234 |

**Table Tags**

| Tag | Description | Example |
|---|---|---|
| Compliance / Sensitivity | Data Governance | classification=restricted - Sensitive PI |
| Life Cycle Management State | Tag that is used to enforce Lifecycle Management. Four possible states:Standard (90 day TTL)Pre Prod, is going through productionisation, do not deleteExtension, TTL has been extended, paired with tag belowExemption, this resource is exempt, paired with tag below. | lcm_state=standardlcm_state=pre-prodlcm_state=extensionlcm_state=exemption |
| Extension Period (Days) | Specifies the extension period in days | lcm_extension_period=30 |
| Exemption Type | Specifies the exemption type | lcm_exemption_type=bu_owned |
| Deletion Date | The date which the asset is expected to be deleted.Visible to user | deletion_date=20251122 |

### Unity Catalog Tag Management and Enforcement

Terraform supports the tagging of clusters, catalogs, schemas, tables, columns, and volumes. When Terraform is being used to create these assets, they should also be tagged via Terraform.

Enforcing tag at creation time for compute can be achieved via compute policies for classic clusters, and budget policies for serverless compute. Both types of policy can be created programmatically.

For Unity Catalog objects, the user will not have the ability to create catalogs. Depending upon the option selected in Unity Catalog design, they may have the option to create schemas and volumes. In both cases they will have the ability to create jobs and tables.

An enforcement mechanism is required for user created objects, as the user may not specify tags correctly during the creation of the object. This will be an automated nightly job that assigns default values to untagged objects. More detail on this can be found in section 6.

Changing tags from default options would be governed by tag policies. These tag policies would prevent general users from changing tags, whilst allowing the DNA team to change as needed.

There exist assets created at runtime that can be tagged and removed with automation jobs, such as notebooks. Notebooks should be archived in an automated fashion to prevent bloat in the workspace. There is not a direct ‘archive’ operation that can be performed by a job, but there are export and delete operations. We recommend deleting stale tables and assets, but archiving the logic (i.e., notebooks, functions, sql queries) required to reproduce them. The DnA team can define an archive space (for example, a workspace folder or another workspace) and apply tags, such as:

- User

- Team

- Original workspace path

- Date Archived

The nightly automation functionality can be extended to permanently delete archived items archived greater than 30 days (or however many the DnA team see fit). When a user submits a request to restore these items with proper justification, the DnA member can restore to the original path.

## Identity Design

Alinta's current identity management implementation demonstrates mature integration with Azure Entra ID through SCIM provisioning patterns, where new Entra groups are created and added to enterprise applications for Databricks access. This approach provides solid foundational security while maintaining alignment with enterprise identity governance standards.

The existing pattern follows a user-centric ownership model where individual users create and own data assets, creating potential operational risks when users are unavailable or leave the organization. Additionally, the current single-catalog architecture requires complex conditional logic for cross-domain access control, increasing administrative overhead and potential security gaps.

Catalog-separated architecture requires fewer specialized toolchain groups because domain boundaries are cleaner and governance patterns are more standardized. The schema-separated approach needs more complex permission management groups to handle the inherent complexity of mixed-domain visibility and access patterns.

### Group Design

For this project, the Entra group design adopted by Alinta has been reviewed and preserved without change. This is because during the workshop discovery process, no pain points relating to the group design eventuated.

### Identity Provider Synchronisation

Presently, Alinta is following the Azure Databricks SCIM provisioning pattern of creating new Entra Groups, and adding these groups to an enterprise application.

There are marginal benefits to be gained from moving from SCIM provisioning to Automatic Identity Management(AIM), however the benefits do not presently justify the effort required to move to AIM. Many of the API calls supporting AIM are still in beta and not battle tested. Nested groups are supported in AIM, but Alinta has not outlined any need for nested groups.

Databricks therefore recommends that Alinta proceeds with SCIM provisioning until a need to use AIM eventuates.

### Entitlements Framework

After further consultation with the DNA team, it was explained that there is not a need for teams within the ‘Pilot for Teams’ environment to share datasets across teams. The only datasets that they would be accessing outside of their team data assets are datasets available within a ‘Shared’ schema. When questioned about the longevity of this assumption, the DNA team was confident that it would hold for at least two years.

Given the above, the entitlements framework will not be required in the immediate term, and its rollout should be delayed.

If the cross team data sharing is required in the future, the entitlements framework should be reassessed.

### Password Storage and Auditing

Presently, there are instances of Alinta personnel storing username and passwords as plaintext within notebooks.

To resolve this issue, Databricks makes the three following recommendations:

- User Education: Providing education to end users about the problems with storing secrets in plain text, as well as education about how to use Databricks Secrets.

- Databricks Secrets: Databricks secrets works by containing containers called *scopes*, to store secrets in as key-value pairs. For example, a scope might be created for the Risk Analytics team called ‘weather-api’. Within this weather-api scope, they store *secrets* that used to access an API they regularly call in their workflows. For example, they might store ‘username’ and ‘password’ within this scope.Secrets are stored in an encrypted database owned and managed by Databricks, and access to these secrets are governed by Access Control Lists (ACLs).

- Detection of Plain Text Passwords: Presently there is not a native solution within Databricks that supports the detection of plain text passwords. We recommend that Alinta consider the adoption of a third party tool to detect these passwords.Alinta are currently using Github to manage the version control of their repos. Github provides Secret Scanning solutions that address this painpoint.

# Onboarding Lifecycle

This section details the proposed onboarding lifecycle for teams and users within the Alinta Databricks Pilot for Teams workspace. It begins by defining what constitutes a "team" from a provisioning perspective, outlining the various components (compute, policies, storage, security, and tags) that will be configured for different team sizes. The document then describes the streamlined onboarding processes for both new teams and individual users, leveraging existing systems like ServiceNow for requests and approvals, and integrating with Entra Group Provisioning and Terraform for automated setup. This structured approach aims to standardize provisioning, reduce manual effort, and ensure consistent application of governance and security policies from the outset.

## What Constitutes a Team?

In the Pilot for Teams workspace, the definition of a team is not standardized. Instead the DNA team performs an assessment of a raised request and provides a solution based upon requirements. Recognizing this process, the below represents what Databricks believes should constitute a provisioning template for teams of various sizes:

- Compute:

  - Classic Shared Cluster - All Purpose compute created for each team (similar to existing setup).

  - Access to Medium Sized Serverless Warehouse (“SQL Endpoint Prod”) - Allows SQL analysts to run workbench type SQL queries. (Similar to existing setup).

- Policies:

  - Cluster policies for classic compute - A cluster policy allowing users to create small personal compute clusters whilst enforcing the assignment of tags to those clusters (already implemented). These cluster policies need to be extended to include the additional proposed tags in section 3.4.1.

  - Serverless Budget Policies (Future) -Serverless Compute is an excellent fit to replace the large shared classic clusters at Alinta, however this should not be implemented until Serverless Budget policies are generally available. Each team will need its own budget policy to assign cost back to the team.

- Storage:

  - As discussed in section 3.3, we recommend teams should be provisioned with UC catalog and default schema, and a single managed volume under the default schema.

- Security:

  - Entra Groups : Teams are defined and granted access through Entra Group. These Groups are passed through to the Databricks account through SCIM provisioning, where they appear as Databricks (external) groups.

  - Databricks Groups: The team Entra Groups are recognized as groups within Databricks, and Unity Catalog governs granting privileges to the team within Databricks.

  - Grants:

    - Catalog

    - Default Schema

    - Default Volume

    - Shared Classic Cluster

    - Use Privileges on Prod Warehouse

    - Cluster Policy

    - Shared Workspace Folder

- Tags:

  - Environment

  - Department

  - Team

  - Sub-team - Where applicable

  - Project / Use Case - Where applicable

  - Cost Center

## Onboarding Process

The Pilot for Teams onboarding framework leverages three core enterprise integration components that work in orchestrated sequence to transform manual provisioning processes into systematic, auditable workflows. ServiceNow serves as the primary workflow orchestration engine, capturing business requirements, routing approval workflows, and maintaining comprehensive audit trails. Azure Entra ID group provisioning acts as the identity management foundation, where approved ServiceNow requests automatically trigger the creation of standardized team groups that flow through SCIM synchronization to establish Databricks access boundaries and role-based permissions. Terraform infrastructure automation completes the provisioning workflow by consuming team configuration metadata to systematically deploy compute resources, storage assets, and governance policies while ensuring consistent resource allocation, security compliance, and cost management across all team environments.

The only change to this process that Databricks is recommending is to tag resources with team tags as described in section 3.4.

## Components

### ServiceNow

The existing ServiceNow process would remain unchanged, as it is successfully addressing its core use case of logging requests and grants for access. Our proposed design would leverage ServiceNow in its current state.

Databricks recommends embedding the ServiceNow request number throughout the process to improve auditability.

### Entra Group Provisioning

Databricks’ design would leverage the existing group provisioning process. Our understanding of this process is documented below:

After receiving a Service Now request, the Cloud Devops team creates the necessary Entra Groups. Once this Entra group has been created, the DNA team assigns the Entra Group to the Enterprise Application created to support SCIM provisioning. These groups are provisioned in the Databricks account every 40 minutes.

### Terraform

The existing Terraform implementation demonstrates solid infrastructure-as-code practices with YAML-based team configuration files. This pattern provides separation of configuration from implementation logic while enabling version-controlled, auditable infrastructure changes.

The Terraform scripts pertaining to Pilot environments are modularized, with the following resources attached to each environment:

name

env_name

schema_name

schema_storage_root

business_unit

workspace

resource_group

storage_account

catalog

catalog_objects_owner

writer

reader

As it stands, it is clear to see the default values and overrides for each Pilot team (or environment).

Databricks recommends enhancing this pattern by applying the tags as outlined in section 3.4.  For instance, metadata fields can be declared in the YAML files that define each environment:

environments:

  - name: CorpFinance

    business_unit: Retail

    # Add metadata fields that could become tags later:

    cost_center: "RC001"

    data_classification: "internal"

    team_contact: "corpfinance@alinta.com"

    retention_days: 90

These tags can be applied to the following resources with Terraform:

- Schemas

- Volumes

- Catalogs

- Clusters

- Workspace Directories

Example of applying custom tags to a resource:

resource "databricks_cluster" "my_cluster" {

  cluster_name           = "example-cluster"

  spark_version          = "14.3.x-scala2.12"

  node_type_id           = "Standard_DS3_v2"

  autotermination_minutes = 60

  num_workers            = 1

  custom_tags = {

    "Environment" = "Development"

    "Project"     = "DataIngestion"

    "Owner"       = "team-data"

  }

}

The team can modularize these custom tags in a similar fashion to the rest of the Terraform code.  To implement this, the team will need to declare tagging-related module variables in /modules/pilot-for-teams-environment/variables.tf to accept tags, environment tags, compliance tags, etc. Next, they must enhance the YAML configuration in pilot-for-teams-environments.yml to include metadata fields (tags) for each team environment. Then, they need to update all resource definitions in the module files to merge the tag variables into resource properties or custom_tags parameters. After, they must modify the module calls in main.tf to pass the YAML metadata as structured tag parameters to each environment module instance.

Here is documentation for use of custom_tags in Terraform Databricks resources.

Runtime-created objects will not be tagged with Terraform and require the Automation jobs detailed in below sections.

These include:

- Tables created by users

- User notebooks

- Personal Clusters

- MLflow experiments / models

- Temporary Views

- User-Defined Functions

# Path to Production

Establishing a clear and consistent path to production is critical for ensuring the reliability, traceability, and governance of all data assets. Different personas — data engineers, data scientists, and analysts — interact with the data platform in distinct ways, but all should follow standardized promotion workflows to move work from development to production environments. As part of this process, it is important to assess the current end-user and developer experience to identify friction points and opportunities for improvement. Evaluating usability, workflow efficiency, and overall developer productivity provides valuable insight into how well the platform supports its users in building, testing, and promoting their work. Based on these findings, actionable recommendations can be made to enhance the overall experience, streamline repetitive tasks, and strengthen consistency across teams.

Having a formal path to production provides clear accountability, governance, and traceability for all data assets, which is difficult to achieve if pilot artifacts are allowed to persist indefinitely in a shadow production state using tags. While tagging can temporarily preserve valuable work, it does not enforce lifecycle policies, quality checks, or versioning standards, and it can lead to unmanaged growth, orphaned artifacts, and inconsistent access controls. Reviewing the current path to production is essential to identify where friction and inefficiencies occur. Applying best practices—such as automated quality gates, standardized review processes, and clear handoff points—can help reduce delays, ensure compliance, and accelerate delivery from development to production. A structured promotion process ensures that only validated, reviewed, and appropriately governed assets are deployed, reducing operational risk, simplifying maintenance, and enabling more reliable collaboration. It also establishes a scalable and repeatable framework that grows with the organization and its data ecosystem while optimizing both administrator and developer experience.

**IT Developers** have been split into two sub-groups, ‘Standard’ and ‘Advanced’.

**Standard IT Developers** should develop ETL pipelines, jobs, and transformations in development workspaces, commit changes to Git, and promote through Dev, UAT, and Production environments using automated deployment pipelines such as Databricks Asset Bundles (DABs).

**Advanced IT Developers :** Teams with advanced software development life cycle (SDLC) experience have the autonomy to develop within the Pilot for Teams workspace using their preferred methods, including their own GitHub Repos and CI/CD processes.

To deploy beyond the pilot environment, these advanced teams must adhere to the standards set by the DNA team. Details on deploying newer Databricks features that may be supported by the standard DABs process can be found in Section 5.5.

**Data scientists** should manage experiments, notebooks, and models in a similar manner, leveraging MLflow for model versioning and incorporating DABs to deploy validated models to production.

**SQL Analysts** (referred to in the requirements as ‘Business Analysts’ ) should develop SQL queries, dashboards, and other BI artifacts in Pilot environments. After handoff, the DNA team will use Git-backed Databricks Repos in conjunction with DABs to promote these assets into governed production workspaces.

All project types — ETL, ML, and BI — should adhere to the same underlying principles: development and testing in isolated environments, automated promotion to production, and comprehensive versioning and metadata management. This approach ensures that production assets are reliable, auditable, and maintainable, while still enabling experimentation and innovation in development and pilot environments.

## IT Developers

IT Developers create DBT code that focuses on the Silver to Gold transformations that power business intelligence, reporting, and downstream analytics use cases. IT Developers build workload features that are redeveloped into production assets. As a result, IT Developers should have read-only access to DEV, UAT and Prod catalogs, and write into the Pilot catalog.

IT Developers building DBT transformations develop in **Dev**** User workspace**, with appropriate tagging and asset management. Developers can promote assets for UAT testing by writing to their team’s schema within the Pilot UAT Catalog. These data assets will be accessible from the Pilot UAT Workspace.

The IT team should have a Git Repo where they develop these workloads in a develop or feature branch. IT Developers should start development in a feature branch with the dbt-sql DABs template or create a customized version. The DBT can be run as a task in a Databricks workflow. This overall job can be defined and deployed with a bundle.

Depending on Alinta processes, IT Developers initiate CI/CD with a ServiceNow ticket and/or a PR. After approval, a DNA team member can pick up the bundle and start productionising through Pipeline workspaces.

Automated CI checks can run:

- dbt compile

- Dbt test

- SQL linting

- Code review approval

Note: When executing a DBT task within a Databricks workflow, the platform automatically generates a temporary authentication token for the user or service principal running the job. This token is injected as an environment variable that DBT profiles can reference to authenticate against the target workspace and SQL warehouse, eliminating the need for manual credential management. For DBT executions outside of Databricks (such as local development or external CI/CD pipelines), explicit authentication is required through OAuth machine-to-machine (M2M) authentication, personal access tokens (PATs), or service principal credentials.

			Figure 3: High Level Path to Production for Standard IT Developers

## IT Developers - Advanced

During discovery with the Merchant and Digital teams, it emerged that there is a type of IT developer with strong experience in Software Development and the associated practices. This type of user may want to use more advanced practices and tools than a typical user.

Generally, these teams should have permission to use these tools unless a compelling justification exists to prevent their use.

Following discovery workshops with the Merchant team, questions were raised regarding the integration of their existing GitHub repositories into the Pilot for Teams environment. A working solution, demonstrated in the diagram below, was developed after reviewing their proposed approach and implementing minor revisions.

Figure 4: High Level Path to Production for Advanced IT Developers

The Pathway to Production for Advanced IT Developers (such as the Merchant Team) is designed to maintain independent CI/CD processes and integrate them with the Databricks environment without impacting the DNA team's operations (see Figure 4).

**CI/CD Process for the Merchant Team:**

- **Independent Repository:** The Merchant team is permitted to maintain its own GitHub repository within the PfT environment.

- **Managed CI/CD:** They manage this repository with their own CI/CD processes, including using GitHub Actions to automatically push new commits from GitHub to Databricks.

- **Production Submission:** When the team is ready for productionization, they create a feature branch in the adh-general repository and populate it with their code.

**Technical Considerations:**

- **VSCode Integration:** While the VSCode to Databricks pathway is established, we recommend utilizing Databricks Connect over the VSCode extension due to reported issues with the extension's robustness.

- **Separate Repositories:** Maintaining a separate GitHub repository poses no issue, as Databricks allows for straightforward linking of different workspace folders to different repositories.

**Key Consideration: Environment Dependencies**

The only anticipated complexity is managing **environment-dependent actions**. Developers must ensure their code executes successfully across different Databricks environments and two distinct GitHub repositories, which may have varying conditions. However, given the team's strong proficiency in Software Development practices, this is not expected to be a significant issue.

## Business Pilots

SQL Analysts should follow a structured template and pathway when developing their business pilots. This allows analysts to innovate and create value using SQL in the pilot workspace, while ensuring that production-grade assets adhere to Alinta's engineering and governance standards. The core of this pathway is a formal handoff and refactoring process, where successful SQL-based pilots are converted into robust, version-controlled DBT models by Alinta’s DNA team before being deployed to production.

This approach empowers BAs to experiment freely while guaranteeing that production assets are maintainable, scalable, and integrated into the central data transformation framework. The promotion workflow is systematic and divided into four distinct phases:

1. Experimentation Phase

During this initial phase, BAs develop insights and prototype logic within their designated pilot catalog using standard SQL queries. The goal is to validate the business value of a potential data product or report before committing to a full-scale production build. To facilitate this process and ensure a degree of consistency, BAs are encouraged to use **standardized templates**. These templates provide a foundational structure for common analytical tasks, such as customer segmentation, time-series analysis, or financial reporting.

- BAs have read access to production data and write access to their team's pilot catalog to create tables and views.

- **Templates** are available in a shared repository and offer pre-written SQL structures that BAs can adapt to their specific needs. This accelerates the development process and introduces a level of standardization even in the experimental phase.

- The final output of this phase is a set of working SQL scripts that successfully transform source data into a valuable output, ready for handoff to the DNA team.

- The process to develop a template that works for Alinta will be iterative (taking 1 - 2 weeks), and can impose requirements such as: one query per transform.

A typical analyst query, adapted from a template during this phase, might look like this:

-- Analyst's prototype query for customer segmentation, adapted from a standard template

CREATE OR REPLACE TABLE pilot.pilot_retail_finance.customer_segments_temp AS

SELECT

  customer_id,

  CASE

    WHEN lifetime_value > 10000 THEN 'High Value'

    WHEN lifetime_value BETWEEN 5000 AND 10000 THEN 'Mid Value'

    ELSE 'Low Value'

  END AS customer_segment

FROM prod.retail_gold.customer_metrics;



2. Validation & Handoff

Once a pilot has proven its value, the BA initiates the productionization process by submitting a formal handoff request through a streamlined Jira ticket. To simplify this process, the ServiceNow ticket will include a **handoff checklist** that the BA must complete. This ensures that the DNA team receives all the necessary information in a clear and concise format.

**ServiceNow Handoff Checklist:**

- **[ ] Business Justification:** A brief summary of the business value and the problem this data product solves.

- **[ ] Final SQL Scripts:** Attach the final, working SQL code.

- **[ ] Source Tables:** List all the source tables and views used in the query.

- **[ ] Expected Output:** Describe the expected output, including the columns, data types, and a sample of the data.

- **[ ] Stakeholders:** List the key business stakeholders for this data product.

This checklist-driven approach simplifies the handoff process for the BA and provides the DNA team with a complete and actionable request.

A highlevel representation of this process is captured in the diagram below:

Figure 5: High Level Path to Production for SQL Analysts

3. Refactoring Phase

Upon receiving the Jira request, a Data Engineer (DE) from the DNA team converts the BA's SQL pilot into a production-grade DBT model. This phase ensures that all production assets are robust, maintainable, and fully integrated into Alinta's data ecosystem.

To streamline this process, the DE uses **sqlglot**, an advanced SQL transpiler, to automatically handle the initial translation of the BA's SQL into the target dialect used in the DBT project. This automated step significantly accelerates the development timeline.

The DE's primary role is to take the output from sqlglot and seamlessly integrate it into the production DBT framework. This involves:

- Replacing hardcoded table names with dynamic source and ref functions to build a resilient data lineage.

- Applying the appropriate configurations for materialization, security, and governance tags directly within the model.

- Ensuring the new model is documented and tested as part of the standard CI/CD deployment pipeline.

By leveraging an automated transpiler, the DE can focus on implementing best practices and ensuring the final data product is reliable and efficient, rather than spending time on manual-syntax conversion.

**Before (BA's SQL):**

-- Provided by the Business Analyst

SELECT

  c.customer_id,

  CASE

    WHEN c.lifetime_value > 10000 THEN 'High Value'

    WHEN c.lifetime_value BETWEEN 5000 AND 10000 THEN 'Mid Value'

    ELSE 'Low Value'

  END AS customer_segment

FROM prod.retail_gold.customer_metrics AS c;



**After (DE's DBT Model -** **models/retail/customer_segments.sql****)****:** *The DE uses**sqlglot**to produce the core logic, then integrates it into a production-ready DBT model.*

-- Refactored into a DBT model by the Data Engineer

{{

  config(

    materialized='table',

    tags=['finance', 'retail']

  )

}}

SELECT

  customer_id,

  CASE

    WHEN lifetime_value > 10000 THEN 'High Value'

    WHEN lifetime_value BETWEEN 5000 AND 10000 THEN 'Mid Value'

    ELSE 'Low Value'

  END AS customer_segment

FROM {{ ref('customer_metrics') }} -- Using a reference to an upstream DBT model



4. Production Integration

Once the logic is refactored into a DBT model and validated, the DE commits the new code to Alinta's Git repository.

- The commit triggers the established CI/CD pipeline, which automatically tests, documents, and deploys the DBT model into the production environment.

- The business pilot is now a governed, observable, and maintainable asset within the production data platform, managed entirely through DBT. The appropriate tags for cost allocation, ownership, and governance are applied within the DBT project configuration.

- This can follow the same DBT standardization as above in the IT Developer section.

An example of the configuration in a dbt_project.yml file might include:

# Example dbt_project.yml configuration

models:

  alinta_dbt:

    retail:

      +materialized: table

      +tags: ['retail']

    finance:

      +materialized: view

      +tags: ['finance', 'pii']



## ML Pathway

The DNA team stressed that the 'Deploy as Code' approach is the required Path to Production for Data Scientists. This directly aligns with the recommendations below, and we want to explicitly confirm that 'Deploy as Code' is the methodology we are proposing.

**Workspace Access Needs**

Data Scientists should develop ML models in the Dev User workspace. Data Scientists should always write to the Pilot Catalogs.

Data Scientists should leverage widgets and DABs variables to parameterize catalog and schema references to avoid manually changing names. This way, there is only one version of the code that is deployed across environments.

The below diagram presents a high-level view of the approach proposed above, combined with the content contained in ‘Starting a ML project with DABs’.

			Figure 6: High Level Path to Production for Data Scientists

**Upskilling Data Scientists to use PySpark**

Data scientists develop models primarily in pandas within the Pilot workspace and then transition to production-ready code in the development environment by refactoring to PySpark with the help of software engineers. This refactoring and handover process is time-intensive and lacks a fully seamless workflow, with minimal ML-specific testing or validation applied during the transition. Data scientists’ access is appropriately limited to User Prod for experimentation, while engineering teams manage Dev -> Prod Pipeline deployments. This setup ensures governance and control but highlights opportunities to streamline collaboration, testing, and code handover practices.

Though refactoring is unavoidable at this time, we recommend training Data Scientists to develop more with PySpark, and there are targeted courses available to address this, such as: https://customer-academy.databricks.com/learn/courses/3486/advanced-machine-learning-with-databricks

In the course, Data Scientists will use Spark for data preparation, model training, and deployment, while also gaining hands-on experience with Spark ML and pandas APIs on Spark. This course will introduce advanced concepts like hyperparameter tuning and scaling Optuna with Spark. This course will use features and concepts introduced in the associate course such as MLflow and Unity Catalog for comprehensive model packaging and governance.

**Starting a ML project with DABs**

There exists a customized MLOps template developed by Fujitsu to fit Alinta’s data science needs. However, adoption is still considered low. This section recommends a further customization to the existing template that can encourage usage.

Develop a custom bundle template that Data Scientists can use to start a project. When instantiating an ML project, Data Scientists will need to have a local copy of the bundle template code. They should then initialize the ML project with the following command:

databricks bundle init <path-to-alinta-custom-template>

Databricks Asset Bundles supports using variables and even complex variables, full documentation here.

Databricks Asset Bundles dependencies for local development are specified in the requirements*.txt file at the root of the bundle project, but job task library dependencies are declared in your bundle configuration files and are often necessary as part of the job task type specification.

The recommendation for installing dependent libraries is to use cluster policies. This allows enforcement of cluster-scoped library installations.

For Runtime 15.0 and above, it is possible to directly list the requirements.txt file as a library dependency for a cluster.

**resources**:

  **jobs**:

    **my_job**:

      # ...

      **tasks**:

        - **task_key**: my_task

          # ...

          **libraries**:

            - **requirements**: ../requirements.txt

            - **whl**: /Workspace/Shared/Libraries/my-wheel-0.0.1-py3-none-any.whl

            - **whl**: /Volumes/main/default/my-volume/my-wheel-0.1.0.whl

For runtime 13.3 and above, init scripts from Volumes can be used to install dependencies.

**Handoff and MLops**

Data Scientists can handoff the ML code to a ML Engineer, who adds CI/CD workflows to deploy the model through Pipeline workspaces. The ML Engineer may need to extract metadata and update tags in the bundle code.

This is the recommended branching strategy and jobs in each stage:

- DevelopmentBundleCI stage

  - onDevelopment job

    - Triggered by PR from feature/* to development

    - Bundle validate and deploy to DEV Pipeline

    - Model training in DEV Pipeline

    - Unit Test in DEV Pipeline

- MainBundleCI stage

  - onMain job

    - Triggered by PR from development to main

    - Bundle validate and deploy to UAT Pipeline

    - If deploy-as-code mode: Model training in UAT PipelineElse: Model copy to UAT Pipeline from DEV Pipeline

    - Integration Test in UAT Pipeline (or additional unit test and model performance test in UAT Pipeline)

    - Data Scientist can validate model results from UAT User

- ProdCD stage

  - onRelease job

    - Create a release branch and waiting for manual reviewer approval

    - Bundle validate and deploy to PROD Pipeline

    - Prod job includes retraining or deep copy

    - Monitoring job is deployed and scheduled

**MLflow experiment tracking from Pipeline**

An existing painpoint is that Data Scientists cannot access MLflow experiments from the other workspaces. This will remain relevant as our recommended target architecture would restrict Data Scientists to the Dev User workspace.

To enable PfT Data Scientists to access their experiment results from the Dev User Workspace,  the recommended approach is to expose MLflow experiment metadata and run details through Unity Catalog tables, enabling Data Scientists to query and visualize experiment results without direct workspace access. This job can be performed by a Service Principal with at least:

- User Workspace access to the specific Pipeline workspace

- CAN_READ on the specified experiment (or all experiments via script).

Create a scheduled Databricks job that extracts MLflow experiment metadata from the Pipeline workspaces and writes into a Unity Catalog table (e.g., in the UAT or PROD catalog). The job can query the MLflow tracking data via API and write summary tables (e.g., experiment name, run_id, metrics, params, tags) into catalog.schema.mlflow_experiment_summary.

This would allow Data Scientists to read the outputs from DEV, UAT or PROD catalog, while maintaining strict workspace isolation and decoupling experiment access from Pipeline workspace privileges.

It may also be helpful to know that there are MLflow system tables, accessible by admins by default.

**Tagging in MLflow**

Some Data Scientists request the ability to isolate their team’s MLflow experiments from others in the Pilot workspace. This can be achieved by tagging the MLflow runs with their team’s tags.

Once this is done, users can enter their team’s tag in the Experiment search bar:

## Newer Databricks Features

Databricks Asset Bundles (DABs) are the recommended method for migrating assets between Databricks environments. It is important to note, however, that newly released Databricks features may initially only support manual deployment (ClickOps), with DAB support typically added in a subsequent release.

As such, the below table specifies some of the newer features in Databricks, and details the currently recommended deployment process.

| No. | Feature | Recommended Deployment Pathway |
|---|---|---|
| 1. | Databricks Apps | Standard DABs process |
| 2. | AI/BI Dashboards | Standard DABs process |
| 3. | AI Gateway | Serverless AI Gateways not yet supported by DABs. Only external models and provisioned throughput endpoints are currently supported. Standard DABs process is recommended where applicable, otherwise ClickOps. |
| 4. | AI Vector Search | Not directly available via DABs, but can be deployed via a notebook, which is supported in DABs. |
| 5. | AI Model Serving | Standard DABs process |
| 6. | Model APIs | Standard DABs process |
| 7. | Lakebase | Standard DABs process |
| 8. | Lakeflow | Generally follows standard DABs process, however some connectors are not yet available. |
| 9. | Genie Spaces | Not presently available via DABs. Recommended approach is deploy input data assets via DABs, and deploy Genie Spaces via ClickOps |
| 10. | DBSQL | Standard DABs process |
| 11. | SQL Alerts | Not yet supported via DABs. Recommended approach is to preserve the alert SQL in DABs, and deploy via ClickOps. |
| 12. | Clean Rooms | Not presently available via DABs. Recommended approach is deploy input data assets via DABs, and deploy Genie Spaces via ClickOps |

Standard DABs Process: Features which are eligible for the standard DABs process can be deployed as code, much like any other resource in DABs. See this link here for more information.

# Automation

For the proposed solution to work with the scale of Alinta, four automation jobs are required to reduce manual handling. All of these jobs should be configured to run as service principles. These will likely leverage the Databricks python sdk.There should be separate audit tables created for each of these jobs to record any operation they have performed.

| Job | Function | Schedule | Output |
|---|---|---|---|
| Tagging | Applies any tags not able to be applied at creation | Daily | Tagged objects |
| Security | Enforcement of Access | Daily | Cleaned Access to data objects |
| Monitoring | Prepare stats about workspace usage | Dependent on Dashboard refreshes | MVs for Observability tools |
| Cleaning | Enforcement of 90 day rule | Daily | Cleaned Environment |

## Tagging Job

Although the tagging of assets created by Terraform can be applied at the time of creation, user created assets will not be tagged. A Databricks Workflow should be configured that lists untagged assets and applies the relevant tag. This job will list objects in the workspace, and apply default tags.

## Security Job

By default, newly created objects list the creator of the object as the owner, granting the owner ‘Manage’ permissions on that object. This allows a user to control access to these data objects and potentially share them with other users. To prevent this, Databricks recommends an automated job to revoke the ‘Manage’ access of the owner, and revoke any access granted to other users. Additionally, this job would grant ‘Can View’ permissions on any job, or ‘Select’ permissions on any table to all other team members in the same team as the creator. This empowers teams to be aware of any data assets used by their team, and to support their ability to make more targeted requests when requesting support from the DNA team.

## Monitoring Job

This would extract relevant information from tagged tables, and the system tables and prepare consolidated tables for performant reporting in dashboards. For example, the following views could be created

- Access Drift: This would be a join of the system.information_schema.table_privilages table and the list of tables tags to identify which individuals have access to tables outside of their team.

- Shadow IT - Comparing scheduled deletion of objects with the creation date to identify long lived data objects that risk becoming data products that have not followed the formal path to production.

- Duplicated effort/data - Combine query data with team tags to identify likely cases of duplicated processes and redundant sources of truth.

## Cleaning Job

This job would enforce the 90 day rule in the Pilot for Teams workspace. This job would examine the LCM status to check if an object was in a non-exempt state, and when it was scheduled for deletion. Finally, it would delete any relevant objects.

*Figure**7**: Flow of Automation Jobs*

# Observability

Observability can be enhanced as a result of tag based approach in Section 3.4. Existing Alinta dashboards in PowerBI can be enhanced by the additional tagging data to provide more granular reporting.

Additionally, Databricks professional services can deploy out of the box dashboards to address common use cases. These dashboards are at no additional cost, apart from the initial setup cost, and the compute to populate the dashboard.

## FinOps

Alinta already has a mature report for FinOps observability. However, the report is limited in how granular it can report, as the current tagging approach does not provide sufficient information.

By passing the additional tagging information such as ‘team’, the dashboards are able to report on how this tag is trending over time.

				*Figure**8**: Reporting on team usage over time*

## SecOps

For details on observability of cross domain access, see section 3.5.3 relating to the entitlements framework and observability.

Leveraging the tag approach and the system tables, reporting on access drift is now possible. Every data asset will belong to a tagged team, and each data asset can be audited to show which users have access.

Any user who has access to assets outside of their team can be reported upon.

## Lifecycle Management Observability.

Databricks recommends constructing a Dashboard to report on the status of objects and where they are in the Lifecycle Management process. This will enable the DNA team to identify and remove potential sources of shadow IT within Alinta. Objects excluded from the 90 day deletion rule can be reported on to allow a quick identification of pseudo data products within the Pilot for Teams workspace.

# Glossary of terms

| Term | Definition |
|---|---|
| ABAC | Attribute-Based Access Control |
| AD | Active Directory |
| ADH | Alinta Data Hub |
| ADLS | Azure Data Lake Storage |
| AIM | Automatic Identity Management |
| ACLs | Access Control Lists |
| APPs | Australian Privacy Principles |
| BI | Business Intelligence |
| CI/CD | Continuous Integration/Continuous Delivery |
| DNA | Data and AI |
| DBT | Data Build Tool |
| FIRB | Foreign Investment Review Board |
| GDPR | General Data Protection Regulation |
| LCM | Lifecycle Management |
| ML | Machine Learning |
| PII | Personally Identifiable Information |
| PoC | Proof of Concept |
| ROI | Return on Investment |
| SIT | System Integration Testing |
| SLA | Service Level Agreement |
| SCIM | System for Cross-domain Identity Management |
| UAT | User Acceptance Testing |
| UC | Unity Catalog |
| YAML | YAML Ain't Markup Language (recursive acronym) |

# Mapping of Requirement to Solution Components

|  | Focus Area / Requirement | Databricks Recommended Solution |
|---|---|---|
| 1 | Data Accessibility - Self-service data exploration | Scaled Governance Solutions |
| 2 | Controlled Collaboration - Secure, shared workspaces | Workspace / Catalog Design |
| 3 | Pathway to Production - Sandbox to production pipeline | Pathway to Production |
| 4 | User Segmentation - Tailored access and permissions by 3 identified roles | Workspace / Catalog Design + Pathway to Production |
| 5 | Governance and Monitoring - Auditable, managed environment | Tagging |
| 6 | Automation - Orchestrated, scalable | Automation |
| 7 | Regulatory Compliance - Aligned to FIRB requirements | Tagging & Automation |
| 8 | Cost Management - Clear cost accountability | Tagging |

# Footnotes

- Addressed in 3.3.1 (Workspace Design), 3.5 (Identity Design), 3.5.3 (Entitlements Framework)

- Addressed in 3.3.1 (Workspace Design)

- Addressed in 3.5 (Identity Design)

- Addressed in 4.3.3 (Terraform)

- Addressed in 3.4 (Tagging Strategy), 6 (Automation)

- Addressed in 3.4 (Tagging Strategy), 6 (Automation)

- Addressed in 3.4 (Tagging Strategy), 6 (Automation)

- Addressed in 7 (Observability)

-  Addressed in 3.3.1 (Workspace Design), 3.5.3 (Entitlements Framework), 4.3.3 (Terraform),  5 (Path to Production)

- Addressed in 5 (Path to Production)

- Addressed in 5 (Path to Production)

- Addressed in 5.2 (Business Pilots)

- Addressed in 5.3 (ML Pathway)

- Addressed in 3.5.4 (Password Storage and Auditing)
