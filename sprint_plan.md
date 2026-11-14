# Two Week Sprint Plan

Sprint goal: deliver a documented, reproducible PySpark pipeline that cleans the supplied healthcare insurance sample data and publishes validated analytical outputs to Redshift.

| Week | Focus | Deliverables | Exit criteria |
|---|---|---|---|
| 1 | Requirements and solution design | SRS, source-to-target mapping, Redshift schema, use-case definitions, Jira backlog, test checklist | Stakeholders agree on definitions, join keys, privacy constraints, and missing-field dependencies |
| 2 | Implementation and testing | S3 ingestion, PySpark cleaning, Redshift writes, output tables, manual QA, presentation | Pipeline runs end-to-end in target environment; row counts and joins reviewed; blocked use cases documented |

## Suggested day-by-day sequence

- Days 1–2: confirm scope, data dictionary, success definitions, and access controls.
- Days 3–4: create schemas, table DDL, and source-to-target mappings.
- Day 5: review documents and Jira stories; approve sprint scope.
- Days 6–7: implement ingestion and cleaning modules.
- Days 8–9: implement analytical outputs and Redshift writes.
- Day 10: run data quality checks, capture result snapshots, demo, and list follow-up data needs.

## Roles

Product owner/business analyst: definitions and acceptance. Data engineer: pipeline and Redshift. QA analyst: test checklist and reconciliation. Scrum lead: Jira, blockers, and review.
