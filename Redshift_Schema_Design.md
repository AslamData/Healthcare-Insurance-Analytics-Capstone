
 Redshift Schema Design

Schemas: `healthcare_clean` and `"project-output"`.

Types and columns below are taken from the Redshift catalog query. The keys are **logical design keys**; verify uniqueness and relationships with data-quality checks. Redshift key declarations are informational.

## Clean tables

| Table | Columns and data types | Logical primary key | Foreign keys |
|---|---|---|---|
| `healthcare_clean.claim` | `claim_id BIGINT`, `patient_id BIGINT`, `disease_name VARCHAR`, `sub_id VARCHAR`, `claim_status VARCHAR`, `claim_type VARCHAR`, `claim_amount NUMERIC`, `claim_date DATE` | `claim_id` | `patient_id` → `patient.patient_id`; `sub_id` → `subscriber.sub_id` |
| `healthcare_clean.disease` | `subgroup_id VARCHAR`, `disease_id BIGINT`, `disease_name VARCHAR` | `disease_id` | `subgroup_id` → `subgroup.subgroup_id` |
| `healthcare_clean.group_subgroup` | `subgroup_id VARCHAR`, `group_id VARCHAR` | `(group_id, subgroup_id)` | `group_id` → `policy_group.group_id`; `subgroup_id` → `subgroup.subgroup_id` |
| `healthcare_clean.hospital` | `hospital_id VARCHAR`, `hospital_name VARCHAR`, `city VARCHAR`, `state VARCHAR`, `country VARCHAR` | `hospital_id` | — |
| `healthcare_clean.patient` | `patient_id BIGINT`, `patient_name VARCHAR`, `patient_gender VARCHAR`, `patient_birth_date DATE`, `patient_phone VARCHAR`, `disease_name VARCHAR`, `city VARCHAR`, `hospital_id VARCHAR` | `patient_id` | `hospital_id` → `hospital.hospital_id` |
| `healthcare_clean.policy_group` | `group_id VARCHAR`, `group_name VARCHAR`, `group_type VARCHAR`, `country VARCHAR`, `premium_written NUMERIC`, `zip_code VARCHAR`, `city VARCHAR`, `reporting_year SMALLINT` | `group_id` (confirm uniqueness) | — |
| `healthcare_clean.subgroup` | `subgroup_id VARCHAR`, `subgroup_name VARCHAR`, `monthly_premium NUMERIC` | `subgroup_id` | — |
| `healthcare_clean.subscriber` | `sub_id VARCHAR`, `first_name VARCHAR`, `last_name VARCHAR`, `street VARCHAR`, `birth_date DATE`, `gender VARCHAR`, `phone VARCHAR`, `country VARCHAR`, `city VARCHAR`, `zip_code VARCHAR`, `subgrp_id VARCHAR`, `elig_ind VARCHAR`, `eff_date DATE`, `term_date DATE` | `sub_id` | `subgrp_id` → `subgroup.subgroup_id` |




## Analytics output tables

These are aggregate or filtered result tables. No primary or foreign key constraints are defined for them; the listed grain describes the expected row identity.

| Table in `"project-output"` | Columns and data types | Expected grain |
|---|---|---|
| `average_monthly_premium` | `average_monthly_premium NUMERIC`, `subscriber_count BIGINT`, `metric_id SMALLINT` | One summary row; `metric_id` |
| `cashless_claims_over_50000` | `claim_id BIGINT`, `patient_id BIGINT`, `claim_amount NUMERIC` | One row per qualifying claim; `claim_id` |
| `claim_count_by_city` | `patient_city VARCHAR`, `claim_count BIGINT` | One row per patient city |
| `disease_claim_count` | `disease_name VARCHAR`, `claim_count BIGINT` | One row per disease |
| `female_over_40_knee_surgery_past_year` | `patient_id BIGINT`, `age INTEGER`, `service_date DATE` | One row per qualifying patient/service event |
| `group_profitability_proxy` | `group_id VARCHAR`, `group_name VARCHAR`, `group_type VARCHAR`, `reporting_year SMALLINT`, `premium_written NUMERIC`, `approved_claim_amount NUMERIC`, `profitability_proxy NUMERIC` | One row per group and reporting year |
| `group_with_max_subgroups` | `group_id VARCHAR`, `subgroup_count BIGINT`, `max_count BIGINT`, `group_name VARCHAR` | One row per group tied for the maximum |
| `hospital_patient_count` | `hospital_id VARCHAR`, `patient_count BIGINT`, `hospital_name VARCHAR`, `city VARCHAR` | One row per hospital |
| `rejected_claim_count` | `rejected_claim_count BIGINT`, `metric_id SMALLINT` | One summary row; `metric_id` |
| `subgroup_subscription_count` | `subgroup_id VARCHAR`, `subscriber_count BIGINT`, `subgroup_name VARCHAR`, `monthly_premium NUMERIC` | One row per subgroup |
| `subscriber_count_by_group_type` | `group_type VARCHAR`, `subscriber_count BIGINT` | One row per group type |
| `subscribers_under_30_with_subgroup` | `sub_id VARCHAR`, `age INTEGER`, `subgrp_id VARCHAR` | One row per subscriber; `sub_id` |
| `under_18_cancer_patients` | `patient_id BIGINT`, `age INTEGER`, `disease_name VARCHAR`, `hospital_id VARCHAR` | One row per qualifying patient/disease |

## Relationship summary

- Claims link to patients and subscribers.
- Patients link to hospitals.
- Subscribers link to subgroups.
- Diseases link to subgroups.
- Groups and subgroups have a many-to-many relationship through `group_subgroup`.
- Output tables are derived results and do not replace the clean source tables.

## Integrity notes

Redshift primary and foreign key declarations are informational. The pipeline must check primary-key uniqueness, orphaned foreign keys, and expected output grain. The cashless-claim and knee-surgery outputs may be empty because the supplied source data lacks the required coverage/procedure details.
