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

> Cross-reference of the clarifying questions in [[Alinta Detailed Design - Catalog and Security#4. Clarifying Questions for Alinta]] against the historical notes from Allison:
> - [Allison Historical Notes 1](https://docs.google.com/document/d/1gGS1Q41fcjDvoMQe6yz9cl3ak5kVcduM6E_0jWGtdlc/edit?usp=sharing)
> - [Allison Historical Notes 2](https://docs.google.com/document/d/1hCpTxLaQbCkH7d5p2J0ulatSxhT4DZq5Gu9bJy0xraM/edit?usp=sharing)

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

**Status:** ⚠️ Partial — Our design has Retail Finance, Energy Supply Tech, Credit Risk, and Corporate Services. Need to confirm whether Trading, Power Generation, and Technology are in scope or deferred.

---

### Q2 — Schema creation autonomy

**Question:** Should teams be able to create their own schemas within their catalog freely, or should new schemas go through a SNOW request to DnA?

**From Allison's notes:** Erik's post-playback feedback: *"no business users in Prod User should have the ability to manage access to even their own data assets (governed via Access Groups)"*. The provisioning model is DnA-controlled via Terraform.

**Status:** ⚠️ Partial — Strong recommendation toward DnA control, but not an explicit Alinta decision. Need to confirm with Andrew/David whether teams get `CREATE SCHEMA` or not.

---

### Q3 — Dev User workspace migration

**Question:** Teams currently develop in Prod User. What's the expected timeline and appetite for moving all pilot development to Dev User?

**From Allison's notes:** All current usage is in Prod User workspace. There was **low engagement and strong pushback** on workspace restructuring in the first playback session, though Erik noted Brad specifically asked for it.

**Status:** ⚠️ Partial — Contentious topic. No timeline agreed. Need to raise with Brad directly.

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

**Status:** ⚠️ Partial — Subject-matter-based access confirmed. Specific isolation mechanism (tags vs. workspace folders) not decided.

---

## Questions With No Answers in Allison's Notes

These need to be raised directly with Alinta:

| # | Question | Topic |
|---|---|---|
| 7 | Catalog for DnA internal work | Catalog Design |
| 8 | Service principal strategy (how many SPs) | Security |
| 9 | Ownership transfer target (group vs SP vs DnA admin) | Security |
| 14 | External API access / network whitelist by team | Security |
| 15 | GitHub Secret Scanning status on ADH repos | Security |
| 17 | Model serving from Pilot catalogs | MLflow |
| 18 | MLflow metadata sync job priority | MLflow |
| 19 | Automation job hosting workspace | Operations |
| 20 | Rollout order for team migration | Operations |
| 21 | Rollback plan / acceptable window | Operations |
| 22 | Communication responsibility for changes | Operations |

---

## References

- [[Alinta Overview]] — Account overview and Allison notes links
- [[Alinta Detailed Design - Catalog and Security]] — Full clarifying questions list
- [Allison Historical Notes 1](https://docs.google.com/document/d/1gGS1Q41fcjDvoMQe6yz9cl3ak5kVcduM6E_0jWGtdlc/edit?usp=sharing)
- [Allison Historical Notes 2](https://docs.google.com/document/d/1hCpTxLaQbCkH7d5p2J0ulatSxhT4DZq5Gu9bJy0xraM/edit?usp=sharing)
