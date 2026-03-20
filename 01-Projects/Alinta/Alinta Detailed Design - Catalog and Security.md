---
type: area
tags:
  - alinta
  - customer
  - detailed-design
  - unity-catalog
  - security
created: 2026-03-19
status: draft
---

# Alinta Detailed Design — Catalog & Security

> Internal working document expanding on the [[Alinta Overview|Alinta Solution Design]] for the Pilot for Teams engagement. This is **not customer-facing**.

---

## 1. Catalog Design

### 1.1 Option Comparison

#### Option A: Catalog per Team + pilot_uat (Recommended)

Each business team gets a dedicated domain-based catalog for development, with a shared `pilot_uat` catalog for structured UAT.

**Structure:**
```
pilot_retail_finance/          ← team catalog (Dev User workspace)
  ├── forecasting/
  ├── billing_improvements/
  └── default/

pilot_energy_supply_tech/
  ├── default/
  └── ...

pilot_credit_risk/
pilot_corporate_services/

pilot_uat/                     ← shared UAT catalog (UAT User workspace)
  ├── retail_finance/
  ├── energy_supply_tech/
  ├── credit_risk/
  └── corporate_services/

prod/                          ← production (read-only from all workspaces)
uat/                           ← BAU UAT (read-only for pilot teams)
```

**Workspace → Catalog Access:**

| Workspace | Write Access | Read Access |
|---|---|---|
| Dev User | `pilot_<team>`, `pilot_uat.<team>` | `prod`, `uat`, `pilot_<team>` |
| UAT User | None | `prod`, `uat`, `pilot_uat.<team>` |
| Prod User | None | `prod` only |

**Pros:**
- Clean lifecycle progression: Dev → UAT → Prod with explicit boundaries
- Strong cost attribution — compute and storage costs isolate by catalog/workspace
- pilot_uat acts as a formal gate before productionisation; prevents "shadow production" in dev
- Simpler Terraform — catalog-level grants instead of complex schema-level conditionals
- No changes required to existing AD groups
- Easier to enforce 90-day cleanup at the catalog level
- Teams own their pilot_uat schema — reduces SNOW ticket overhead for UAT access
- Clear audit trail: who promoted what, when, into pilot_uat

**Cons:**
- Additional workspace usage — Dev User becomes the primary dev workspace (previously Prod User was used)
- More catalogs to manage (one per team + pilot_uat)
- Teams must explicitly promote assets to pilot_uat — adds a step vs. current workflow
- pilot_uat schema ownership by teams could drift if not governed (teams granting access to others)
- If a pilot needs cross-team data during UAT, the pattern is less clear (team A reads from pilot_uat.team_b?)
- Migration effort: existing assets in Pilot catalog need to move to new team catalogs AND the dev workspace

---

#### Option B: Catalog per Team (No Dedicated UAT Catalog)

Each team gets a dedicated domain-based catalog. UAT is handled within the same catalog or via tagging.

**Structure:**
```
pilot_retail_finance/
  ├── dev_forecasting/
  ├── dev_billing/
  ├── uat_forecasting/        ← UAT schemas within same catalog
  └── default/

pilot_energy_supply_tech/
pilot_credit_risk/
pilot_corporate_services/

prod/
uat/
```

**Workspace → Catalog Access:**

| Workspace | Write Access | Read Access |
|---|---|---|
| Dev User / Prod User | `pilot_<team>` | `prod`, `uat` |
| UAT User | None (or read-only to `pilot_<team>.uat_*`) | `prod`, `uat` |

**Pros:**
- Fewer catalogs to manage — no separate pilot_uat catalog
- Simpler mental model: one catalog = one team, everything lives there
- No explicit promotion step — UAT schemas sit alongside dev schemas
- Easier cross-team collaboration: grant schema-level access within same catalog
- Less migration effort than Option A (no pilot_uat provisioning)
- Teams have full autonomy over their catalog structure (schemas for dev, uat, shared, etc.)

**Cons:**
- Dev and UAT assets share a catalog — harder to enforce access boundaries between dev and UAT users
- Business users doing UAT may accidentally access dev schemas if grants aren't tight
- Cost attribution between dev and UAT is blurred (same catalog)
- No formal "promotion" step — risk of UAT becoming a dumping ground
- Schema naming conventions must be strictly enforced (dev_ vs uat_ prefixes) — convention over enforcement
- Harder to clean up after UAT — no clear "revoke access + delete from pilot_uat" pattern
- UAT User workspace grant design is more complex (need schema-level grants within team catalogs)
- Audit trail is weaker: harder to distinguish "this was promoted for UAT" vs. "this is still in dev"

---

### 1.2 Recommendation

**Proceed with Option A.** The explicit pilot_uat catalog provides a stronger governance boundary and a clearer lifecycle signal. The additional operational cost (one more catalog, explicit promotion step) is justified by the audit, cost attribution, and cleanup benefits.

### 1.3 Note: Potential Option D — Per-Domain UAT Catalogs

> [!note] Placeholder — not an official option
> This is an alternative worth revisiting if the shared `pilot_uat` catalog shows scaling issues or if teams grow significantly.

**Idea:** Instead of a single shared `pilot_uat` catalog with per-team schemas, give each domain its own UAT catalog:

```
pilot_retail_finance/              ← team dev catalog
pilot_uat_retail_finance/          ← team UAT catalog
  ├── forecasting/                 ← schemas carried over from dev
  └── billing_improvements/

pilot_energy_supply_tech/
pilot_uat_energy_supply_tech/
  └── ...
```

**What this gains over Option A:**
- Consistent isolation model — catalog-level boundaries in both dev and UAT (no mixed catalog/schema pattern)
- Stronger inter-team isolation at UAT — one team's UAT work cannot interfere with another's
- More granular UAT cost attribution per domain
- Full team autonomy over their UAT catalog — no shared namespace coordination

**What this trades away vs Option A:**
- Catalog proliferation — doubles UAT catalogs (4-5 teams → 8-10 total catalogs), grows with each new team
- DnA loses single-pane-of-glass visibility into all UAT activity (spread across multiple catalogs)
- Cross-domain UAT harder — if a pilot spans two teams, need cross-catalog grants instead of cross-schema within `pilot_uat`
- More Terraform plumbing — each new domain needs both dev and UAT catalog provisioned

#### Deeper Design Considerations

**1. Permission model consistency.** The move to domain-based catalogs in dev (Option A) eliminates the need for schema-level grants — teams get catalog-level access via a single Terraform module. However, in `pilot_uat`, Option A reintroduces the same schema-level grant pattern: each team needs `USE SCHEMA`, `SELECT`, and `MODIFY` on `pilot_uat.<team>`, while being denied access to other teams' schemas within the same catalog. Option D avoids this inconsistency entirely — the same Terraform module and catalog-level grants used in dev apply identically in UAT. The permission model is uniform across all environments.

**2. Multi-project schema segregation.** In Option A, when a team runs multiple projects through UAT simultaneously, all of them share a single schema (`pilot_uat.retail_finance`). This forces table-naming conventions to distinguish between projects (e.g., `forecasting_customer_segments` vs `billing_customer_segments`) — convention over enforcement. In Option D, the team gets a full UAT catalog with per-project schemas that mirror dev exactly:

```
Option A — shared pilot_uat (team with 2 active UAT projects):
pilot_uat/
  └── retail_finance/                    ← single schema for all projects
        ├── forecasting_customer_segments    ← naming convention separates projects
        ├── forecasting_model_output
        ├── billing_invoice_summary
        └── billing_anomaly_flags

Option D — per-domain UAT catalog (same team, same projects):
pilot_uat_retail_finance/
  ├── forecasting/                       ← mirrors dev schema exactly
  │     ├── customer_segments
  │     └── model_output
  └── billing_improvements/              ← mirrors dev schema exactly
        ├── invoice_summary
        └── anomaly_flags
```

Option D preserves the same schema structure in UAT as in dev, meaning promotion scripts and notebook references require no path rewriting beyond the catalog name.

**3. Discoverability and information leakage.** In Option A, all teams require `USE CATALOG` on `pilot_uat` (see Section 2.3.1). This means any team member can discover other teams' schema names within `pilot_uat`, even without data access. While schema names alone are low-sensitivity, they reveal what projects other teams are working on — which may be undesirable for teams running sensitive or commercially confidential pilots. Option D provides complete catalog-level isolation: teams cannot see or discover other teams' UAT catalogs at all.

**4. Access revocation lifecycle.** When a team completes UAT and their pilot is either promoted to production or decommissioned, the cleanup path differs:
- **Option A:** Revoke schema-level grants (`USE SCHEMA`, `SELECT`, `MODIFY` on `pilot_uat.<team>`), drop the schema, but the team retains `USE CATALOG` on `pilot_uat` (since it's shared and other teams still use it). The team can still see remaining schemas in the catalog.
- **Option D:** Revoke catalog-level access to `pilot_uat_<team>`, drop the catalog. Complete removal — no residual access, no stale visibility. Cleaner audit trail for access lifecycle.

**Current assessment:** For Alinta's current scale (~4-5 teams) and short-lived UAT cycles, the shared `pilot_uat` (Option A) is simpler and gives DnA better oversight. However, Option D is a genuinely strong alternative — not just a "phase 2" thought — and becomes the preferred choice if any of the following prove true:

- **Teams regularly run multiple projects through UAT concurrently**, making schema-level separation within `pilot_uat` awkward and convention-dependent
- **The number of teams grows beyond ~5**, at which point the schema-level grant management in `pilot_uat` becomes operationally burdensome and the Terraform module divergence between dev (catalog grants) and UAT (schema grants) adds maintenance overhead
- **Schema-level permission management in `pilot_uat` causes operational friction** — e.g., SNOW tickets for schema access, grant drift, or audit findings about teams seeing other teams' schema names

The recommendation remains Option A for the initial rollout, but the design should be structured so that migrating a team from `pilot_uat.<team>` to `pilot_uat_<team>` is a low-effort operation if Option D becomes warranted.

---

### 1.4 Naming Conventions

#### Catalogs

| Pattern | Example | Purpose |
|---|---|---|
| `pilot_<domain>` | `pilot_retail_finance` | Team development catalog |
| `pilot_uat` | `pilot_uat` | Shared UAT catalog |
| `prod` | `prod` | Production (existing) |
| `uat` | `uat` | BAU UAT (existing) |

Rules:
- Lowercase, underscores only
- Domain names align to business unit / team names in Entra
- No abbreviations unless universally understood (e.g., `corp` for Corporate)

#### Schemas

| Context | Pattern | Example |
|---|---|---|
| Team dev work | `<project_name>` | `pilot_retail_finance.forecasting` |
| Default (migration landing) | `default` | `pilot_retail_finance.default` |
| Legacy migration | `pilot_<team>` (preserve old name) | `pilot_retail_finance.pilot_retailcredit` |
| UAT | `<team>` | `pilot_uat.retail_finance` |

Rules:
- One schema per project/initiative within a team catalog
- Default schema receives migrated objects (preserving old schema name minimises disruption)
- Teams cannot create schemas in `pilot_uat` — only DnA or Terraform provisions these

#### Volumes

| Context | Pattern | Example |
|---|---|---|
| Team default volume | `<team>_files` | `pilot_retail_finance.default.retail_finance_files` |
| Project-specific volume | `<project>_files` | `pilot_retail_finance.forecasting.forecasting_files` |

Rules:
- Managed Volumes only (no External Volumes for PfT teams)
- One volume per schema by default
- Users get `READ VOLUME` + `WRITE VOLUME` — not `CREATE VOLUME`

#### Tables and Views

| Pattern | Example |
|---|---|
| `<descriptive_name>` | `pilot_retail_finance.forecasting.customer_segments` |
| Temp/scratch prefix | `tmp_<name>` or `scratch_<name>` |

Rules:
- No personal prefixes (e.g., `john_test_table`) — the 90-day cleanup handles ephemeral assets
- `tmp_` prefix tables get a 30-day TTL instead of 90

#### Models (MLflow)

| Context | Pattern | Example |
|---|---|---|
| Registered model | `<team>_<model_name>` | `pilot_retail_finance.default.retail_churn_model` |
| Experiment | Tagged with `team`, `project` | `team=retail_finance, project=churn` |

---

### 1.5 Migration Plan

**From:** Single `pilot` catalog → **To:** Domain-based `pilot_<team>` catalogs

#### Phase 1: Provision New Catalogs (Week 1)
- [ ] Create `pilot_retail_finance`, `pilot_energy_supply_tech`, `pilot_credit_risk`, `pilot_corporate_services` catalogs via Terraform
- [ ] Create `pilot_uat` catalog with team schemas via Terraform
- [ ] Create default schema + default volume in each team catalog
- [ ] Apply catalog-level tags (environment, department, team, cost_center)
- [ ] Apply grants to existing Entra groups (see Section 2)

#### Phase 2: Migrate Existing Assets (Week 2–3)
- [ ] For each team schema in the old `pilot` catalog:
  - Create a schema with the **same name** in the new team catalog (e.g., `pilot.pilot_retailcredit` → `pilot_retail_finance.pilot_retailcredit`)
  - Use `CREATE TABLE ... AS SELECT` or `DEEP CLONE` to copy tables
  - Re-register any MLflow models pointing to old catalog paths
  - Update any jobs/notebooks with new 3-level namespace references
- [ ] Validate row counts and checksums post-migration
- [ ] Notify teams of new catalog paths

#### Phase 3: Cutover (Week 4)
- [ ] Redirect all team access from `pilot` to `pilot_<team>`
- [ ] Set `pilot` catalog to read-only (revoke all write grants)
- [ ] Monitor for any queries still hitting old paths (via query history / system tables)
- [ ] Run find-and-replace on notebooks: `pilot.pilot_<team>` → `pilot_<team>.pilot_<team>`
- [ ] Alternatively, add `USE CATALOG pilot_<team>` block to scripts using 2-level namespace

#### Phase 4: Deprecation (Week 6+)
- [ ] After 2 weeks with no traffic to old `pilot` schemas, archive and drop
- [ ] Remove old Terraform config for single Pilot catalog
- [ ] Update documentation and onboarding templates

---

## 2. Security & Data Permissions

### 2.1 Principles

1. **Least privilege by default** — teams get the minimum grants needed for their persona
2. **Catalog-level isolation** — cross-team access is not granted unless explicitly requested and approved
3. **Ownership revocation** — the security automation job revokes individual `MANAGE` on user-created objects
4. **No plaintext secrets** — Databricks Secrets with scoped ACLs; detection via GitHub Secret Scanning
5. **Convention over configuration where possible** — but enforce at the grant level, not just by naming

### 2.2 Entra Group → Databricks Group Mapping

Existing Entra groups are preserved. No new groups are required for the catalog migration.

| Entra Group | Databricks Group | Purpose |
|---|---|---|
| `GG-Azure-DataHub-Pilot-RetailCredit-DEV` | Same (via SCIM) | Dev access to `pilot_retail_finance` |
| `GG-Azure-DataHub-Pilot-EnergySupplyTech-DEV` | Same (via SCIM) | Dev access to `pilot_energy_supply_tech` |
| `GG-Azure-DataHub-Pilot-CreditRisk-DEV` | Same (via SCIM) | Dev access to `pilot_credit_risk` |
| `GG-Azure-DataHub-Pilot-CorpServices-DEV` | Same (via SCIM) | Dev access to `pilot_corporate_services` |
| `GG-Azure-DataHub-DnA-Admin` | Same (via SCIM) | DnA admin — full control |

**Notes:**
- SCIM syncs every 40 minutes from Entra → Databricks account
- No nested groups needed (AIM deferred per original design)
- If new teams are onboarded, a new Entra group is created following the existing pattern

### 2.3 Unity Catalog Grant Matrix

#### 2.3.1 Catalog-Level Grants

| Catalog | Group | Privileges |
|---|---|---|
| `pilot_<team>` | Team DEV group | `USE CATALOG`, `USE SCHEMA`, `CREATE SCHEMA` |
| `pilot_<team>` | DnA Admin | `ALL PRIVILEGES` |
| `pilot_<team>` | Other teams | **No access** |
| `pilot_uat` | All team DEV groups | `USE CATALOG` |
| `pilot_uat` | DnA Admin | `ALL PRIVILEGES` |
| `prod` | All team DEV groups | `USE CATALOG` (read path only) |
| `uat` | All team DEV groups | `USE CATALOG` (read path only) |

#### 2.3.2 Schema-Level Grants

**Team Development Schemas** (`pilot_<team>.<schema>`):

| Group | Privileges |
|---|---|
| Team DEV group | `USE SCHEMA`, `CREATE TABLE`, `CREATE VIEW`, `CREATE FUNCTION`, `CREATE MATERIALIZED VIEW`, `CREATE MODEL`, `CREATE VOLUME` (only on default schema — see Volume rules) |
| DnA Admin | `ALL PRIVILEGES` |

> **Note:** `CREATE VOLUME` is restricted. Teams get it on the default schema only. Additional volumes are provisioned via Terraform/DnA request.

**pilot_uat Schemas** (`pilot_uat.<team>`):

| Group | Privileges | Context |
|---|---|---|
| Team DEV group | `USE SCHEMA`, `CREATE TABLE`, `CREATE VIEW`, `SELECT` | Write access for promoting assets |
| Business UAT users (same team) | `USE SCHEMA`, `SELECT` | Read-only for UAT validation |
| DnA Admin | `ALL PRIVILEGES` | |
| Other teams | **No access** | |

**Production / UAT Catalogs** (`prod.*`, `uat.*`):

| Group | Privileges |
|---|---|
| All team DEV groups | `USE SCHEMA`, `SELECT` |
| DnA Admin | `ALL PRIVILEGES` |

#### 2.3.3 Table & View Grants

Tables and views inherit from schema grants. Additional explicit grants:

| Object | Group | Privileges | Notes |
|---|---|---|---|
| All tables in `pilot_<team>` | Team DEV group | `SELECT`, `MODIFY` | Users can read/write tables in their own catalog |
| All tables in `prod` | All teams | `SELECT` | Read-only |
| All tables in `pilot_uat.<team>` | Team DEV group | `SELECT`, `MODIFY` | For promotion |
| All tables in `pilot_uat.<team>` | Business UAT group | `SELECT` | Read-only UAT |
| User-created tables | Creator | `MANAGE` → **revoked by security job** | See Section 2.5 |

**Materialized Views:**
- Same grants as tables
- Teams can create MVs in their own catalog schemas
- MVs in `pilot_uat` should be avoided — promote base tables, not MVs

#### 2.3.4 Volume Grants

| Volume | Group | Privileges |
|---|---|---|
| Team default volume | Team DEV group | `READ VOLUME`, `WRITE VOLUME` |
| Project volumes (if created by DnA) | Team DEV group | `READ VOLUME`, `WRITE VOLUME` |
| Any volume | Team DEV group | **Not** `CREATE VOLUME` (except default schema) |
| prod volumes | All teams | `READ VOLUME` |

**Key rule:** Users cannot create volumes freely. This prevents proliferation. Volumes are provisioned as part of team/project onboarding via Terraform.

#### 2.3.5 Model & MLflow Grants

| Object | Group | Privileges | Notes |
|---|---|---|---|
| Registered models in `pilot_<team>` | Team DEV group | `CREATE MODEL`, `USE SCHEMA` | Models live in team catalog |
| Specific model | Team DEV group | `APPLY TAG`, `EXECUTE` | Can tag and serve |
| Specific model | Creator | `MANAGE` → **revoked by security job** | Ownership transferred to team group or service principal |
| Models in `prod` | All teams | `EXECUTE` (if serving) | Read/inference only |
| MLflow experiments | Tagged by team | N/A (workspace-level, not UC) | Isolation via tags + search filters |

**MLflow experiment access across workspaces:**
- Data Scientists in Dev User cannot see Pipeline workspace experiments directly
- Solution: scheduled job extracts MLflow metadata → writes to UC table (e.g., `pilot_<team>.default.mlflow_experiment_summary`)
- Team can query their own experiment results without Pipeline workspace access

#### 2.3.6 Function Grants

| Object | Group | Privileges |
|---|---|---|
| UDFs in `pilot_<team>` | Team DEV group | `CREATE FUNCTION`, `EXECUTE` |
| UDFs in `prod` | All teams | `EXECUTE` |

### 2.4 Persona → Permission Summary

Consolidated view across all object types:

#### Business Analyst

| Object | Dev User | UAT User | Prod User |
|---|---|---|---|
| `pilot_<team>` catalog | USE, CREATE TABLE/VIEW, SELECT, MODIFY | — | — |
| `pilot_uat.<team>` | SELECT | SELECT | — |
| `prod` | SELECT | SELECT | SELECT |
| `uat` | SELECT | SELECT | — |
| Volumes | READ, WRITE (team) | — | — |
| Models | — | — | — |
| Functions | EXECUTE | — | — |

#### Data Scientist

| Object | Dev User | UAT User | Prod User |
|---|---|---|---|
| `pilot_<team>` catalog | USE, CREATE TABLE/VIEW/MODEL, SELECT, MODIFY | — | — |
| `pilot_uat.<team>` | SELECT, MODIFY (promotion) | SELECT | — |
| `prod` | SELECT | SELECT | SELECT |
| `uat` | SELECT | SELECT | — |
| Volumes | READ, WRITE (team) | — | — |
| Models | CREATE, EXECUTE, APPLY TAG | — | EXECUTE (serving) |
| MLflow experiments | Full (own team, via tags) | Read (via UC table) | — |

#### IT Developer (Standard)

| Object | Dev User | UAT User | Prod User |
|---|---|---|---|
| `pilot_<team>` catalog | USE, CREATE TABLE/VIEW/FUNCTION, SELECT, MODIFY | — | — |
| `pilot_uat.<team>` | SELECT, MODIFY (promotion) | SELECT | — |
| `prod` | SELECT | SELECT | SELECT |
| `uat` | SELECT | SELECT | — |
| Volumes | READ, WRITE (team) | — | — |
| Models | — | — | — |
| Functions | CREATE, EXECUTE | — | EXECUTE |

#### IT Developer (Advanced)

Same as Standard IT Developer, plus:
- May use own GitHub repos with CI/CD into Dev User workspace
- Must follow DnA standards for productionisation via adh-general repo
- No additional UC grants beyond Standard

#### DnA Admin

| Object | All Workspaces |
|---|---|
| All catalogs | `ALL PRIVILEGES` |
| All schemas | `ALL PRIVILEGES` |
| All objects | `ALL PRIVILEGES` |
| Terraform | Full provisioning access |

### 2.5 Security Automation Job — Detailed Design

#### Purpose
Enforce ownership and access policies on user-created objects that cannot be controlled at creation time.

#### Schedule
Daily, 2:00 AM AEST (off-peak)

#### Service Principal
`sp-pilot-security-job` with `ALL PRIVILEGES` on all pilot catalogs

#### Logic

```
FOR each pilot_<team> catalog:
  FOR each schema in catalog:
    FOR each object (table, view, model, function, volume) in schema:

      # 1. Revoke individual MANAGE
      IF object.owner is a user (not a group or SP):
        TRANSFER ownership to team DEV group
        LOG action to audit table

      # 2. Revoke ad-hoc grants
      FOR each grant on object:
        IF grantee is not in [team_dev_group, dna_admin_group, sp-pilot-security-job]:
          REVOKE grant
          LOG action to audit table

      # 3. Ensure team visibility
      IF team_dev_group does NOT have SELECT on object:
        GRANT SELECT to team_dev_group
        LOG action to audit table

      # 4. Tag compliance (delegates to tagging job, but flag here)
      IF object is missing required tags:
        FLAG in audit table for tagging job
```

#### Audit Table

Location: `pilot_admin.security.audit_log` (admin catalog, not accessible to teams)

| Column | Type | Description |
|---|---|---|
| `event_id` | STRING | UUID |
| `event_timestamp` | TIMESTAMP | When the action was taken |
| `catalog_name` | STRING | |
| `schema_name` | STRING | |
| `object_type` | STRING | TABLE, VIEW, MODEL, FUNCTION, VOLUME |
| `object_name` | STRING | |
| `action` | STRING | TRANSFER_OWNERSHIP, REVOKE_GRANT, GRANT_SELECT, FLAG_UNTAGGED |
| `previous_owner` | STRING | |
| `new_owner` | STRING | |
| `grantee` | STRING | Who had the grant revoked/added |
| `privilege` | STRING | e.g., MANAGE, SELECT |
| `reason` | STRING | Policy rule that triggered the action |

### 2.6 Controlling Data Sources

To prevent teams from onboarding new data sources without DnA oversight:

#### Restricted Privileges (NOT granted to teams)

| Privilege | Why Restricted |
|---|---|
| `CREATE EXTERNAL LOCATION` | Prevents teams from mounting arbitrary ADLS paths |
| `CREATE STORAGE CREDENTIAL` | Prevents teams from registering new cloud credentials |
| `CREATE MANAGED STORAGE` | Storage locations are DnA-managed only |
| `EXTERNAL USE LOCATION` | Prevents bypass of UC governance via external access |
| `CREATE EXTERNAL TABLE` | All tables should be Managed Tables within team catalogs |
| `CREATE CONNECTION` | Federated query connections are DnA-provisioned only |

#### Network Policies
- Notebook internet access restricted via network policies
- Specific API endpoints can be whitelisted per team request
- Egress rules enforced at the workspace level

### 2.7 Secret Management

#### Scope Design

| Scope | Owner | Example Secrets |
|---|---|---|
| `<team>-<project>` | Team DEV group | API keys, DB passwords, service credentials |
| `dna-admin` | DnA Admin | Infrastructure credentials, SP tokens |

#### ACLs

| Scope | Principal | Permission |
|---|---|---|
| `<team>-<project>` | Team DEV group | `READ` |
| `<team>-<project>` | DnA Admin | `MANAGE` |
| `dna-admin` | DnA Admin | `MANAGE` |

#### Plaintext Detection
- GitHub Secret Scanning enabled on all repos
- Custom patterns added for Databricks PATs (`dapi*`), Azure connection strings
- Nightly scan of workspace notebooks for patterns matching credentials (regex-based, logged to audit)

### 2.8 Cross-Team Access (Future)

Currently not required (DnA confirmed 2-year horizon). When needed:

1. Team A requests access to Team B's schema via SNOW ticket
2. DnA reviews and grants `SELECT` on specific schema only (not catalog-level)
3. Grant is time-bound (90 days default) and tracked in audit table
4. Security job revokes expired cross-team grants automatically
5. Entitlements framework (ABAC) reassessed at that point

---

## 3. Open Items & Decisions Needed

- [ ] Confirm final list of team domains and catalog names with Alinta
- [ ] Confirm pilot_uat schema ownership model — team-owned vs DnA-owned
- [ ] Decide on `tmp_` table TTL — 30 days vs. match the standard 90
- [ ] Confirm service principal naming and provisioning for automation jobs
- [ ] Determine if MLflow experiment sync job is in scope for this phase
- [ ] Network policy whitelist — get list of required external endpoints from teams
- [ ] Confirm cross-team access is truly deferred (re-validate with DnA)

---

## 4. Clarifying Questions for Alinta

### Catalog Design

1. **Team domain list** — Can you confirm the final list of business teams that will each get a dedicated catalog? We have Retail Finance, Energy Supply Tech, Credit Risk, and Corporate Services so far — are there others planned or any that should be combined?
2. **Schema creation autonomy** — Should teams be able to create their own schemas within their catalog freely, or should new schemas go through a SNOW request to DnA? (This affects whether we grant `CREATE SCHEMA` to teams or reserve it for Terraform.)
3. **Dev User workspace migration** — Teams currently develop in Prod User. What's the expected timeline and appetite for moving all pilot development to Dev User? Is a parallel running period needed where both workspaces are active?
4. **pilot_uat ownership** — Who should own the schemas in `pilot_uat` — the team themselves (faster UAT turnaround, less SNOW overhead) or DnA (tighter control)? If team-owned, are you comfortable with teams managing their own UAT access grants?
5. **Cross-team data in UAT** — If a team's pilot depends on another team's pilot output during UAT, how should that be handled? Options: (a) copy the dependency into `pilot_uat`, (b) grant cross-schema read in `pilot_uat`, (c) disallow and require prod data only.
6. **Existing shared schema** — There's currently a "Shared" schema in the Pilot catalog. Should this become its own catalog (`pilot_shared`), live as a schema under an existing catalog, or be deprecated?
7. **Catalog for DnA internal work** — Does the DnA team need their own pilot catalog for internal tooling, automation development, or testing? Or do they operate exclusively in Pipeline workspaces?

### Security & Permissions

8. **Service principal strategy** — How many service principals does Alinta want to manage? One per automation job (tagging, security, cleaning, monitoring) or a single shared SP for all automation? Are there existing SPs we should reuse?
9. **Ownership transfer target** — When the security job revokes individual `MANAGE` on user-created objects, should ownership transfer to (a) the team's Entra group, (b) a dedicated service principal, or (c) the DnA admin group?
10. **Notification on revocation** — When the security job revokes a user's ad-hoc grant or transfers ownership, should the user be notified (e.g., email, Teams message)? Or is the audit log sufficient?
11. **Existing ad-hoc grants** — Are there known cases today where users have granted access to objects outside their team? Should we do a one-time audit and cleanup before going live, or let the security job handle it on first run?
12. **Secret scopes** — Do teams currently have any Databricks Secret scopes set up? If so, what's the naming convention and who manages them? We want to avoid breaking existing workflows.
13. **Plaintext credentials** — How widespread is the plaintext password issue in notebooks? Is it isolated to a few teams or across the board? This affects whether we prioritise detection tooling or user education first.
14. **External API access** — Which teams need outbound internet access from notebooks (e.g., calling external APIs, weather data, market feeds)? We need a whitelist to configure network policies.
15. **GitHub Secret Scanning** — Is GitHub Secret Scanning already enabled on the ADH repos, or does this need to be set up? Are there other repos outside of ADH that Pilot teams use?

### MLflow & Models

16. **MLflow experiment isolation** — Is experiment tagging by team sufficient for isolation, or do Data Scientists want workspace-level separation (e.g., separate experiment folders per team)? The current doc mentions DS teams wanting separate workspaces for this.
17. **Model serving from Pilot** — Will any team need to serve models directly from their pilot catalog (e.g., for prototyping a real-time endpoint), or is model serving strictly a production concern?
18. **MLflow metadata sync** — Is the proposed job to sync MLflow experiment metadata from Pipeline workspaces into UC tables a priority for this phase, or can it be deferred?

### Operations & Rollout

19. **Automation job hosting** — Where should the automation jobs (tagging, security, cleaning, monitoring) run — in a Pipeline workspace, a dedicated admin workspace, or the Dev User workspace?
20. **Rollout order** — Is there a preferred order for migrating teams to the new catalog structure? Should we start with the smallest team (lowest risk) or the most bloated (highest value)?
21. **Rollback plan** — If a team experiences issues post-migration, what's the acceptable rollback window? Should we keep the old Pilot catalog read-only for 2 weeks, 4 weeks, or longer?
22. **Communication** — Who is responsible for communicating changes to the business teams — DnA, or should we prepare materials for DnA to distribute?

---

## 5. References

- [[Alinta Overview]] — Account overview and context
- [Final Solution Design](https://docs.google.com/document/d/1xHKisvagYV7-4s3_tnhAwhiHCPhIBGoVr5cthrnb9mU/edit?tab=t.0) — Source document this design expands on
- [Final Presentation Deck](https://docs.google.com/presentation/d/1Jd8ZDHcErYeLiRxl8SQoS4Gvt-GvY6kyLQd6Y6Mgqs8/edit?slide=id.g380036ad87e_0_282#slide=id.g380036ad87e_0_282)
