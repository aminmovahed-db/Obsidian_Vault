---
source: https://docs.google.com/document/d/1hCpTxLaQbCkH7d5p2J0ulatSxhT4DZq5Gu9bJy0xraM/edit
type: google-doc
captured: 2026-03-21
description: Allison/Billy/Erik brainstorm session notes (2025-09-22) — Alinta Pilot for Teams sync with Gemini-generated notes
---

## Billy / Erik / Allison Alinta brainstorm session and sync

Invited

Attachments

Meeting records

### Summary

Erik Seefeld provided a comprehensive overview of the project's end-to-end narrative, detailing the provisioning process, governance strategies including enforced tagging, and the importance of observability for cost and usage tracking. Allison Chau discussed MLOps stack structure and the need to simplify the path to production for ML, while Billy O'Connor sought clarification on security scope, to which Erik emphasized defining and enforcing access rights. The key talking points included clarifying the number of workspaces, ensuring a structured design document, outlining automation jobs, and streamlining user experience, especially for ML workflows.



### Details

- **End-to-End Storytelling and Deliverables** Erik Seefeld highlighted the challenge of presenting a cohesive end-to-end narrative for the project's various components, including provisioning, governance, and observability. They outlined the need to clarify the number of workspaces through discussions with Brad and Andrew, and emphasized the importance of a well-structured design document for workspaces (00:00:00). Erik also mentioned their use of ChatGPT to clarify flow, aiming to unify disparate documentation elements into a coherent whole (00:01:18).

- **Provisioning Process** Erik Seefeld detailed the provisioning flow, starting with a service now request, followed by an approval workflow, an automation trigger, and a CI/CD pipeline in Azure DevOps using Terraform. This pipeline provisions the catalog, schemas, clusters, cluster policies, and server warehouses for each team, with all elements subsequently tagged (00:02:22). Erik also explained that the automation flow should include the creation of Entra groups for security (00:03:20).

- **Governance and Automation Jobs** Erik Seefeld discussed the governance piece, emphasizing the importance of an enforced tagging strategy via an automation job (00:03:20). They suggested a high-level table to outline automation jobs, including their function, schedule, and output, with examples such as enforcing tagging, security, usage monitoring, and cleaning (00:04:35). Erik noted that four main types of automation jobs are required to support the onboarding, governance, and maintenance stories (00:05:55).

- **Observability and User Experience** Erik Seefeld explained that the final piece of the project narrative is observability, which will involve understanding current practices and implementing FinOps V2 dashboards for cost and usage tracking. They also highlighted the importance of secops for monitoring who is accessing resources and recommended exploring existing observability solutions, such as those implemented by Brendan at Queensland Energy (00:06:57). Erik noted that the path to production and user experience, especially for ML, would be a key focus for Allison Chau (00:08:12).

- **Design Granularity and Terraform Structure** Erik Seefeld clarified that while some aspects like the onboarding process require detailed design, the overall approach should focus on high-level design, avoiding pseudo-code (00:09:15). They provided recommendations for Terraform repository organization, suggesting a structure with a folder per team containing variables for their specific resources (catalog, schemas, clusters, SQL warehouses). Erik also offered to share internal Terraform resources and an example clean layout from another customer (APA) to guide the team (00:10:11) (00:23:11).

- **Workspace and Catalog Design Recommendations** Allison Chau shared that current workspace and catalog designs may not require radical changes, as existing structures are largely effective (00:13:44). Erik Seefeld agreed, suggesting presenting three options for workspace design but maintaining a strong opinion that separate catalogs are the recommended Option A, even if schema-based approaches are allowed as alternatives (00:14:54). Erik also emphasized the importance of a detailed tagging strategy that covers both initial creation and ongoing enforcement (00:24:11).

- **MLOps and Path to Production** Allison Chau presented on the MLOps stack project structure, noting that data engineers typically own CI/CD configurations for ML code. Erik Seefeld stressed the need to be prescriptive about simplifying the path to production for ML, particularly in the context of Fujitsu's ML framework (00:16:09). They highlighted the challenge of the current "front door process" for moving ML models into production and suggested exploring ways to make collaboration between data and end-users smoother (00:17:43). Allison also confirmed their intent to investigate how to improve the pair programming and refactoring process within the ML workflow (00:18:48).

- **Security and Access Provisioning** Billy O'Connor sought clarification on the scope of security and secops. Erik Seefeld explained that security is implicitly part of governance, focusing on defining groups (e.g., read/write) for catalogs and enforcing those rights. They emphasized the need for an auditable and easy way for users to request and be granted access, suggesting that while initial access might be via Terraform, a sidecar process with an interface (like a table in Databricks) would be better for ongoing, auditable access management (00:25:11).



### Suggested next steps

- Erik Seefeld will help Billy O'Connor with the provisioning and observability automation stuff.

- Erik Seefeld will send a sample contents page for the flow of the document.

- Erik Seefeld will log Allison Chau into APA to show a clean layout for Terraform and put Allison Chau and Billy O'Connor in a chat with Chavi and Steve for Terraform best practices.

- Billy O'Connor will create a high-level table for the automation jobs section, including their purpose, schedule, and output, and talk to Erik Seefeld about how they currently look at everything for observability.

- Allison Chau will talk to Fujitsu about their ML framework and changes/improvements to facilitate DABs and easier collaboration.

- Allison Chau will get someone to walk her through the ML parts to discuss unit and ML-specific testing, and look at DAB's code for recommendations on standardizing work.

- Billy O'Connor will get some idea of what they do for observability today.



*You should review Gemini's notes to make sure they're accurate.* *Get tips and learn how Gemini takes notes*

*Please provide feedback about using Gemini to take notes in a* *short survey.*
