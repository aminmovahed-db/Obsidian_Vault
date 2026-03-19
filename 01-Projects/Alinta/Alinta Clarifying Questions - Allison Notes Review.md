---
type: area
tags:
  - alinta
  - customer
  - clarifying-questions
  - review
created: 2026-03-19
status: draft
---

# Clarifying Questions — Allison Notes Review

> Cross-reference of the clarifying questions in [[Alinta Detailed Design - Catalog and Security#4. Clarifying Questions for Alinta]] against Allison's historical notes and forwarded emails.
>
> **Sources reviewed:**
> - [Allison Historical Notes 1](https://docs.google.com/document/d/1gGS1Q41fcjDvoMQe6yz9cl3ak5kVcduM6E_0jWGtdlc/edit?usp=sharing)
> - [Allison Historical Notes 2](https://docs.google.com/document/d/1hCpTxLaQbCkH7d5p2J0ulatSxhT4DZq5Gu9bJy0xraM/edit?usp=sharing)
> - Email: Pilot for Teams - Data Science Notes (Andrew Davis, 12 Aug 2025)
> - Email: Pilot for Teams Uplift - DnA Requirements (Andrew Davis, 14 Aug 2025) — has 3 attachments (PDF workshop board, User Requirement Summary docx, DnA Workshop Notes docx) not yet reviewed
> - Email: Pilot for Teams Uplift - Merchant Tech (Andrew Davis, 12 Aug 2025)
> - Email: Pilot for Teams Uplift - Digital Follow-up notes (Andrew Davis, 12 Aug 2025)
> - Email: Pilot for Teams Uplift - ML Ops Walkthrough (Andrew Davis, 4 Aug 2025)

---

## Questions With Answers (or Partial Answers)

### Q1 — Team domain list

**Question:** Can you confirm the final list of business teams that will each get a dedicated catalog?

**From Allison's notes:** The Confluence excerpt lists domains broader than the 4 in our design:
- Retail (MM, C&I)
- Trading & Portfolio Management
- Power Generation & Development
- Corporate Services (Finance, P&C, HS&E, Legal)
- Technology

Workshop notes also mention Retail Insights, Merchant, and Credit Risk as specific teams.

**From emails:**
- *Merchant Tech email:* Duminda's team (Energy/Trading) has an in-house gas forecasting application they want to expand into Python/Databricks. They have a DFM platform integration and need service accounts for pipeline deployment. This confirms Merchant/Energy Trading as an active team.
- *Digital Follow-up email:* Digital team (Bala) wants sandbox for generative AI use cases (retail customer data, website content). Fab's team needs campaign data management. Bala explicitly requested **separate sandbox environments for different teams**. Andrew committed to early access for the Digital team.
- *Data Science Notes email:* Dan and Todd are data scientists doing ML models (forecasting, classification) — appear to sit within Retail/IT.

**Status:** ⚠️ Partial — The emails confirm at least **6 distinct teams** with active requirements: Retail Finance, Energy Supply Tech / Merchant, Credit Risk, Corporate Services, Digital, and DnA itself. Our design currently only covers 4. Need to confirm whether Digital and Merchant Tech get their own catalogs or sit under an existing domain.

---

### Q2 — Schema creation autonomy

**Question:** Should teams be able to create their own schemas within their catalog freely, or should new schemas go through a SNOW request to DnA?

**From Allison's notes:** Erik's post-playback feedback: *"no business users in Prod User should have the ability to manage access to even their own data assets (governed via Access Groups)"*. The provisioning model is DnA-controlled via Terraform.

**From emails:**
- *Data Science Notes email:* Dan highlighted *"the growing complexity and messiness of managing a large number of tables and models within a single schema"* and the *"lack of personal schemas."* He suggested a dual-schema approach (prototype and pilot) per team. This implies teams want some schema flexibility but within guardrails.

**Status:** ⚠️ Partial — Strong recommendation toward DnA control, but DS teams feel constrained by single-schema setups. A middle ground might be: DnA provisions initial schemas via Terraform, but teams can request additional project schemas via SNOW. Need to confirm with Andrew/David.

---

### Q3 — Dev User workspace migration

**Question:** Teams currently develop in Prod User. What's the expected timeline and appetite for moving all pilot development to Dev User?

**From Allison's notes:** All current usage is in Prod User workspace. There was **low engagement and strong pushback** on workspace restructuring in the first playback session, though Erik noted Brad specifically asked for it.

**From emails:**
- *ML Ops Walkthrough email:* Explicitly states *"Parallel Sets of Environments — Pipeline workspace (for engineers) and user workspace (for data scientists). Workloads should run in user workspace so data scientists can access ML flow metrics."* This confirms the workspace separation model is understood and desired.
- *Data Science Notes email:* Dan described needing a development pipeline and the difficulties of running models across environments — *"effective iteration on productionised models requires the ability to work within a development pipeline."*

**Status:** ⚠️ Partial — Contentious topic. No timeline agreed. However, MLOps email confirms the user workspace / pipeline workspace split is the desired target architecture. Need to raise migration timeline with Brad directly.

---

### Q4 — pilot_uat ownership

**Question:** Who should own the schemas in `pilot_uat` — the team themselves or DnA?

**From Allison's notes:** Workshop notes: *"Automated reporting would be sufficient instead of automated enforcement"* and Andrew wanted time-bound access. The governance stance leans toward DnA control.

**Status:** ⚠️ Partial — No explicit decision. Lean is toward DnA ownership given governance posture.

---

### Q5 — Cross-team data in UAT

**Question:** If a team's pilot depends on another team's pilot output during UAT, how should that be handled?

**From Allison's notes:** Acknowledged as a gap: *"Cross-business access needs to be solved for"* — but no approach was decided.

**Status:** ⚠️ Partial — Known problem, no solution chosen.

---

### Q6 — Existing shared schema

**Question:** There's currently a "Shared" schema in the Pilot catalog. Should this become its own catalog, a schema, or be deprecated?

**From Allison's notes:** David mentioned: *"Common schema — Date Dimensions, Weather Data"*. Shared data exists but no decision on its future home.

**Status:** ⚠️ Partial — Confirmed shared data exists. No decision on structure.

---

### Q10 — Notification on revocation

**Question:** When the security job revokes a user's ad-hoc grant or transfers ownership, should the user be notified?

**From Allison's notes:** *"Perhaps a notification to say 'you've been using for greater x days, consider moving to production'"* and *"Automated reporting would be sufficient instead of automated enforcement."*

**Status:** ✅ Likely answered — Alinta prefers notifications, not silent enforcement. Design should include user notification on revocation.

---

### Q11 — Existing ad-hoc grants

**Question:** Are there known cases today where users have granted access to objects outside their team?

**From Allison's notes:** *"Principle of Least Privilege: Many users may have broader access than needed."*

**Status:** ⚠️ Partial — Confirms ad-hoc grants exist broadly, but no cleanup plan discussed.

---

### Q12 — Secret scopes

**Question:** Do teams currently have any Databricks Secret scopes set up?

**From Allison's notes:** David said: *"we can't give people outside of team access to the pipeline workspace, because there are secret scopes which we do not want them to have."* Beyza flagged: *"Need a way to support users storing their passwords without using plain text. Databricks backed secrets would be a good fit."*

**Status:** ✅ Answered — Secret scopes exist in Pipeline workspaces. David is protective of them. Plaintext passwords in notebooks is a known problem — Databricks-backed secrets recommended for Pilot users.

---

### Q13 — Plaintext credentials

**Question:** How widespread is the plaintext password issue in notebooks?

**From Allison's notes:** Beyza confirmed it's a problem and recommended Databricks-backed secrets. Scope/severity not quantified.

**Status:** ⚠️ Partial — Problem confirmed, severity unknown. Need to quantify before deciding detection vs. education priority.

---

### Q16 — MLflow experiment isolation

**Question:** Is experiment tagging by team sufficient for isolation, or do Data Scientists want workspace-level separation?

**From Allison's notes:** *"Should data scientist access to schemas depend on subject matter? Yes. E.g, Retail group, Merchant, Corporate."*

**From emails:**
- *ML Ops Walkthrough email:* *"Workloads should run in user workspace so data scientists can access ML flow metrics"* — DS teams want visibility into MLflow from user workspace. Each model is *"a different folder within the ADH science repo."*
- *Data Science Notes email:* Dan and Todd use SKLearn, Keras, and time series libraries. They work in a single schema and want better organisation. Models are developed in pandas and don't follow Spark standards.

**Status:** ⚠️ Partial — Subject-matter-based access confirmed. DS teams want MLflow access from user workspace (not pipeline). Folder-per-model in repo suggests folder-based isolation is natural. Tags alone may not be sufficient — consider workspace-level experiment folders per team.

---

## New Answers From Forwarded Emails

### Q8 — Service principal strategy

**Question:** How many service principals does Alinta want to manage? One per automation job or a single shared SP?

**From emails:**
- *Merchant Tech email:* Duminda's team explicitly needs *"a service account to run the pipelines"* and *"a service account to deploy from the pipeline."* Andrew and Erik were tasked with creating a *"service account with appropriate restrictions to enable pipeline deployment."*
- *ML Ops Walkthrough email:* The ML framework uses CI/CD pipelines with deployment jobs — implies existing SPs for the engineering pipeline.

**Status:** ⚠️ Partial — At least one team (Merchant/Energy) already needs a dedicated service account for pipeline deployment. Engineering has existing CI/CD SPs. Still need to confirm whether Alinta wants one SP per automation job (tagging, security, cleaning, monitoring) or a shared SP for all.

---

### Q14 — External API access / network whitelist

**Question:** Which teams need outbound internet access from notebooks?

**From emails:**
- *Digital Follow-up email:* Bala's team wants to *"run jobs to scrape data, such as new website pages"* and explore *generative AI use cases* with customer data and website content. Needs GPU access and potentially external model APIs.
- *Merchant Tech email:* Duminda's team integrates with the DFM platform (external system) and their own GitHub repository. Compute runs in Databricks with output going back to DFM.
- *Data Science Notes email:* Dan uses SKLearn, Keras, and time series libraries — likely needs PyPI access for package installation.

**Status:** ⚠️ Partial — At least Digital (web scraping, Gen AI APIs), Merchant Tech (DFM integration, GitHub), and Data Science (PyPI packages) need outbound access. Still need a complete whitelist from all teams.

---

### Q17 — Model serving from Pilot

**Question:** Will any team need to serve models directly from their pilot catalog?

**From emails:**
- *Merchant Tech email:* Duminda wants compute to run in Databricks with output integrated back into DFM. This is batch inference, not real-time serving.
- *ML Ops Walkthrough email:* Model promotion follows a CI/CD pipeline (dev → UAT → prod). Andrew expressed *"reservations about suitability for citizen data scientists"* using the full framework. Model serving is a production concern in their current architecture.
- *Data Science Notes email:* Todd described pilot schema as *"quickly deliver unfinished products to business users for early-stage UAT."* This is prototype sharing, not serving.

**Status:** ⚠️ Partial — Current patterns are batch inference and prototype sharing, not real-time serving. Likely safe to defer model serving from Pilot, but should confirm with Andrew that no team plans a real-time endpoint in Pilot.

---

### Q18 — MLflow metadata sync priority

**Question:** Is the proposed job to sync MLflow experiment metadata from Pipeline workspaces into UC tables a priority?

**From emails:**
- *ML Ops Walkthrough email:* *"Workloads should run in user workspace so data scientists can access ML flow metrics."* DS teams explicitly want MLflow visibility.
- *Data Science Notes email:* Dan needs *"the ability to work within a development pipeline"* and cited difficulties running models across environments.

**Status:** ⚠️ Partial — DS teams clearly want MLflow access, which validates the need for the sync job. However, if workloads run in the user workspace (as MLOps email suggests), DS teams may have direct access to experiments — reducing the need for a sync job. Depends on whether Pilot teams run experiments in Pipeline or User workspace.

---

### Q20 — Rollout order for team migration

**Question:** Is there a preferred order for migrating teams?

**From emails:**
- *Digital Follow-up email:* Andrew committed to *"providing early access to sandbox environments"* for the Digital team. Bala explicitly requested early access.

**Status:** ⚠️ Partial — Digital team was promised early access. Could be a natural first candidate for the new catalog structure — smaller scope (Gen AI POC) and explicit interest. Still need to confirm with Andrew if this aligns with the catalog migration rollout.

---

### Q22 — Communication responsibility

**Question:** Who is responsible for communicating changes to business teams?

**From emails:**
- All 5 emails originate from **Andrew Davis** (Delivery Manager, Data & AI). He organises all stakeholder sessions, distributes AI-generated meeting notes, and captures follow-up actions. He is clearly the communication hub.

**Status:** ⚠️ Partial — Andrew is the de facto communication owner for all stakeholder interactions. Still worth confirming whether DnA (Andrew/David) will handle change communication or if we need to prepare materials for them to distribute.

---

## Questions Still Unanswered

These need to be raised directly with Alinta — no information found in Allison's notes or the forwarded emails:

| # | Question | Topic |
|---|---|---|
| 7 | Catalog for DnA internal work | Catalog Design |
| 9 | Ownership transfer target (group vs SP vs DnA admin) | Security |
| 15 | GitHub Secret Scanning status on ADH repos | Security |
| 19 | Automation job hosting workspace | Operations |
| 21 | Rollback plan / acceptable window | Operations |

> **Note:** The DnA Requirements email (14 Aug 2025) has 3 unreviewed attachments: a PDF workshop board export, a User Requirement Summary (.docx), and DnA Workshop Notes (.docx). These may contain additional answers — particularly for Q7 (DnA catalog) and Q9 (ownership transfer).

---

## References

- [[Alinta Overview]] — Account overview and Allison notes links
- [[Alinta Detailed Design - Catalog and Security]] — Full clarifying questions list
- [Allison Historical Notes 1](https://docs.google.com/document/d/1gGS1Q41fcjDvoMQe6yz9cl3ak5kVcduM6E_0jWGtdlc/edit?usp=sharing)
- [Allison Historical Notes 2](https://docs.google.com/document/d/1hCpTxLaQbCkH7d5p2J0ulatSxhT4DZq5Gu9bJy0xraM/edit?usp=sharing)
