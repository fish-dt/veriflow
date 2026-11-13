# ADR-0003: Domain Ownership and Governance

Status: Accepted for MVP

## 1. Template ownership

Veriflow supports two template scopes:

- GLOBAL: maintained by the Veriflow platform.
- INSTITUTION: maintained by an authorized institution administrator.

Institution-owned templates may be created independently or derived from a global template.

A published template version is immutable.

Institution customization must never silently modify a global template.

## 2. Policy ownership

Validation policies are institution-scoped.

A policy may be associated with one or more compatible document types.

Policy versions are immutable after publication.

Each verification run records the exact policy version used.

## 3. Reference-data governance

Reference-data versions follow:

DRAFT -> VALIDATED -> APPROVED -> PUBLISHED

Only authorized users may approve and publish versions.

Published versions are immutable.

An update creates a new version.

Import metadata, source, validation errors, approver and publication time must be retained.

## 4. Human review and overrides

Authorized reviewers may resolve findings and record a final review disposition.

Overriding an automated FAIL requires:
- Reviewer authorization.
- A mandatory reason.
- A recorded resolution.
- An immutable audit event.

A reviewer may also reopen or flag an automated PASS.

The automated result must remain preserved separately from the human review disposition.

A review resolution does not rewrite historical extraction or validation results.

## 5. External application correlation

The external financial institution owns its application and customer records.

Veriflow stores external references for correlation.

Multiple submissions may share the same external application reference.

An external application reference is not a Veriflow-owned customer or application master record.

The MVP does not require Veriflow to create or manage the external application itself.

## 6. Submission boundary

One submission represents exactly one uploaded document.

Each submission has its own:
- Document object.
- Template version.
- Processing lifecycle.
- Extraction runs.
- Validation runs.
- Findings.
- Review history.

Multiple submissions can be associated through an external application reference.

## 7. Historical reproducibility

Every completed verification run must identify:
- Template version.
- Policy version.
- Reference-data versions.
- Extraction engine version.
- Validation engine version.

New configuration versions must not mutate historical results.
