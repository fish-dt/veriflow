# Veriflow — PostgreSQL Data Model

Status: Proposed — pending engineering review

## 1. Database conventions

- PostgreSQL.
- UUID primary keys.
- snake_case table and column names.
- timestamptz for timestamps.
- JSONB only for genuinely variable structured metadata.
- Foreign keys for relational integrity.
- Explicit institution ownership for tenant-scoped records.
- Published configuration versions are immutable.
- Store document binaries in MinIO, not PostgreSQL.

## 2. Tenant and identity

### institutions
- id UUID PK
- name
- code
- status
- created_at
- updated_at

Unique: code.

### users
- id UUID PK
- email
- display_name
- password_hash
- status
- created_at
- updated_at

### institution_memberships
- id UUID PK
- institution_id UUID FK
- user_id UUID FK
- status
- created_at

Unique: institution_id + user_id.

### roles
- id UUID PK
- institution_id UUID FK, nullable for platform roles
- name
- description

### membership_roles
- membership_id UUID FK
- role_id UUID FK

Unique: membership_id + role_id.

Identity design must support institution isolation and platform administration without granting cross-tenant access by default.

## 3. Document templates

### document_templates
- id UUID PK
- institution_id UUID FK, nullable for GLOBAL templates
- template_key
- name
- document_type
- scope: GLOBAL or INSTITUTION
- status
- created_at
- updated_at

Constraint:
- GLOBAL requires institution_id IS NULL.
- INSTITUTION requires institution_id IS NOT NULL.

### template_versions
- id UUID PK
- template_id UUID FK
- version_number
- status: DRAFT, PUBLISHED, DEPRECATED
- layout_definition JSONB
- field_definitions JSONB
- page_constraints JSONB
- created_by UUID FK
- published_by UUID FK, nullable
- published_at
- created_at

Unique: template_id + version_number.

Published versions are immutable.

The JSON definitions contain fixed-layout coordinates, field keys, data types, required status and extraction configuration.

## 4. Validation policies

### validation_policies
- id UUID PK
- institution_id UUID FK
- policy_key
- name
- description
- status
- created_at

Unique: institution_id + policy_key.

### policy_versions
- id UUID PK
- policy_id UUID FK
- version_number
- status: DRAFT, PUBLISHED, DEPRECATED
- rules JSONB
- created_by UUID FK
- published_at
- published_by UUID FK, nullable
- created_at

Unique: policy_id + version_number.

Policies may support multiple compatible document types. Their compatibility and rule structure must be validated before publication.

## 5. Reference data

### reference_data_sets
- id UUID PK
- institution_id UUID FK
- set_key
- name
- description
- owner
- source_description
- status
- created_at

Unique: institution_id + set_key.

### reference_data_versions
- id UUID PK
- reference_data_set_id UUID FK
- version_number
- status: DRAFT, VALIDATED, APPROVED, PUBLISHED
- source_version
- import_metadata JSONB
- effective_from
- effective_to
- created_by UUID FK
- approved_by UUID FK, nullable
- published_at
- created_at

Unique: reference_data_set_id + version_number.

### reference_data_items
- id UUID PK
- reference_data_version_id UUID FK
- canonical_code
- display_label
- description
- status
- effective_from
- effective_to

Unique: reference_data_version_id + canonical_code.

### reference_data_aliases
- id UUID PK
- reference_data_item_id UUID FK
- alias_value
- normalized_alias
- source
- created_at

Aliases must be checked for ambiguity within their applicable reference-data version.

## 6. Submissions and documents

### submissions
- id UUID PK
- institution_id UUID FK
- external_application_reference
- external_customer_reference, nullable
- document_type
- template_version_id UUID FK
- status
- verification_outcome, nullable
- created_by UUID FK, nullable for API integrations
- created_at
- updated_at
- completed_at

Indexes:
- institution_id + created_at
- institution_id + external_application_reference
- institution_id + status

The external application reference is not unique. Several submissions may share it.

### document_objects
- id UUID PK
- submission_id UUID FK UNIQUE
- storage_bucket
- storage_key
- original_filename
- media_type
- file_size_bytes
- sha256_checksum
- upload_status
- uploaded_at
- created_at

The UNIQUE constraint on submission_id enforces one document object per submission.

The original file is stored in MinIO.

## 7. Processing

### processing_jobs
- id UUID PK
- submission_id UUID FK
- job_type
- status: PENDING, PROCESSING, COMPLETED, FAILED, CANCELLED
- attempt_count
- maximum_attempts
- available_at
- lease_expires_at
- claimed_by
- last_error_code
- last_error_message
- created_at
- updated_at

Indexes:
- status + available_at
- lease_expires_at
- submission_id + created_at

### extraction_runs
- id UUID PK
- submission_id UUID FK
- processing_job_id UUID FK
- template_version_id UUID FK
- engine_version
- status
- started_at
- completed_at
- error_code

### field_observations
- id UUID PK
- extraction_run_id UUID FK
- field_key
- raw_value
- normalized_candidate
- confidence
- observation_status
- page_number
- bounding_box JSONB
- evidence_metadata JSONB
- created_at

Index: extraction_run_id + field_key.

Raw extracted values must remain distinguishable from normalized candidates.

## 8. Validation

### validation_runs
- id UUID PK
- submission_id UUID FK
- extraction_run_id UUID FK
- policy_version_id UUID FK
- validation_engine_version
- outcome: PASS, FAIL, REVIEW
- started_at
- completed_at

### validation_run_reference_versions
- validation_run_id UUID FK
- reference_data_version_id UUID FK

Composite primary key: validation_run_id + reference_data_version_id.

This records exactly which reference-data versions were used.

### findings
- id UUID PK
- validation_run_id UUID FK
- finding_code
- category
- severity
- message
- field_key, nullable
- evidence_metadata JSONB
- resolution_status
- created_at
- resolved_at

A finding is an explanation of a result, not an extraction observation.

## 9. Human review

### review_tasks
- id UUID PK
- submission_id UUID FK
- assigned_to UUID FK, nullable
- status
- resolution
- resolution_reason
- created_at
- assigned_at
- completed_at

### review_events
- id UUID PK
- review_task_id UUID FK
- actor_id UUID FK
- event_type
- reason
- comment
- created_at

A review resolution never overwrites the original automated validation outcome.

## 10. Audit and integrations

### audit_events
- id UUID PK
- institution_id UUID FK
- actor_id UUID FK, nullable
- action
- entity_type
- entity_id UUID
- correlation_id
- event_data JSONB
- occurred_at

Append-only. Application permissions must not allow ordinary update or delete operations.

### webhook_endpoints
- id UUID PK
- institution_id UUID FK
- endpoint_url
- secret_reference
- subscribed_events JSONB
- status
- created_at

### webhook_deliveries
- id UUID PK
- webhook_endpoint_id UUID FK
- event_id UUID
- status
- attempt_count
- next_attempt_at
- last_http_status
- last_error
- created_at
- delivered_at

Use idempotency keys for event delivery.

## 11. Integrity requirements

1. Every tenant-owned record must be associated with an institution.
2. A submission must reference a valid template version.
3. A document object must belong to exactly one submission.
4. A validation run must reference the extraction run belonging to the same submission.
5. The selected policy version and reference-data versions must remain identifiable.
6. A finding must belong to a validation run.
7. A review task must belong to a submission.
8. Published configuration versions must be immutable.
9. Audit records must not be silently edited or deleted.
10. Database transactions must protect lifecycle transitions.

## 12. Design questions before SQL migrations

- Should platform users and institution users share one identity table?
- Should template definitions use JSONB or normalized field-definition tables?
- Should policy rules use JSONB with a strict schema?
- Should review tasks support reassignment and multiple review rounds?
- What retention and deletion policies apply to original documents and extracted PII?
- Should webhook secrets be held in an external secret manager or encrypted application storage?

These questions must be resolved before finalizing migrations.
