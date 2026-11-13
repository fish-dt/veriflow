# Veriflow — Domain Model

## 1. Modeling principles

- A submission represents exactly one document.
- A submission is the primary unit of processing and verification.
- A submission may carry an external application reference supplied by an integrating system.
- Veriflow does not own the external institution's customer or application master records.
- Templates, policies, and reference-data versions are immutable once published.
- Every verification result records the exact configuration versions used.
- Extraction observations are separate from business findings.
- A human review resolution does not erase the original automated result.

## 2. Core domain concepts

### Institution

Represents a tenant organization using Veriflow.

Owns or configures:
- Document templates.
- Reference-data sets.
- Validation policies.
- Users and permissions.
- Integration settings.

### Submission

Represents one document submitted for verification.

Important attributes:
- Submission identifier.
- Institution identifier.
- External application reference, optional.
- External customer reference, optional.
- Document type.
- Template version.
- Submission status.
- Verification outcome.
- Creation and completion timestamps.

A submission does not contain multiple uploaded documents.

### Document Object

Represents the stored file associated with a submission.

Important attributes:
- Object identifier.
- Submission identifier.
- Original filename.
- Media type.
- File size.
- Cryptographic checksum.
- Storage object key.
- Upload timestamp.
- Storage status.

The actual binary is stored in MinIO.

### Document Template

Defines how a particular fixed-layout document is processed.

A template has published versions.

A version defines:
- Document type.
- Supported page count or page constraints.
- Expected page dimensions and orientation.
- Field definitions.
- Field coordinates or extraction regions.
- Field data types.
- Required or optional status.
- Extraction configuration.
- Quality-check regions.

Published versions are immutable.

### Field Definition

Defines an expected field in a template.

Examples:
- Full name.
- Date of birth.
- Identification number.
- Issue date.
- Expiry date.
- Address.

A field definition may specify:
- Stable field key.
- Display label.
- Data type.
- Required status.
- Normalization strategy.
- Extraction region.
- Validation constraints.

### Processing Job

Represents asynchronous document processing work.

Important attributes:
- Job identifier.
- Submission identifier.
- Job type.
- Status.
- Attempt count.
- Maximum attempts.
- Lease expiry.
- Available-at timestamp.
- Error classification.
- Created and updated timestamps.

### Extraction Run

Represents one execution of document processing.

Important attributes:
- Extraction run identifier.
- Submission identifier.
- Engine version.
- Template version.
- Processing status.
- Started and completed timestamps.

A retry creates a new run or an explicitly recorded attempt; it must not silently overwrite prior evidence.

### Field Observation

Represents what the document engine observed.

Important attributes:
- Field key.
- Raw extracted value.
- Normalized candidate value, when applicable.
- Confidence score.
- Page number.
- Bounding box or region.
- Evidence reference.
- Extraction method.
- Observation status.

An observation is not a business decision.

### Reference Data Set

Represents a governed collection of accepted values for a particular concept.

Examples:
- Country codes.
- Currency codes.
- Document types.
- Institution-defined branch codes.
- Accepted customer categories.

A set contains immutable published versions.

### Reference Data Item

Represents a canonical value within a reference-data version.

Important attributes:
- Canonical code.
- Display label.
- Description.
- Status.
- Effective-from date.
- Effective-to date, optional.
- Source metadata.

### Reference Data Alias

Maps an alternative representation to a canonical item.

Examples:
- Abbreviations.
- Alternate spellings.
- Legacy codes.
- Known OCR variants.

Aliases must not silently resolve ambiguous values.

### Validation Policy

Defines business checks applied to a submission.

A policy version may define:
- Required fields.
- Accepted value constraints.
- Reference-data matching requirements.
- Date and format checks.
- Cross-field consistency checks.
- Conditions that require human review.

### Validation Run

Represents the evaluation of extraction observations against a specific policy and reference-data snapshot.

It records:
- Submission identifier.
- Extraction run identifier.
- Policy version.
- Reference-data versions.
- Validation engine version.
- Outcome.
- Evaluation timestamp.

### Finding

Represents an actionable validation or processing result.

Examples:
- Required field missing.
- Unreadable document.
- Unsupported document layout.
- Reference value not recognized.
- Ambiguous reference match.
- Expired document.
- Inconsistent field values.

A finding contains:
- Finding code.
- Severity.
- Explanation.
- Related field keys.
- Evidence references.
- Resolution status.

### Review

Represents human examination of a submission or finding.

Important attributes:
- Reviewer.
- Review status.
- Assigned owner.
- Resolution.
- Reason.
- Comments.
- Timestamps.

The original automated result remains preserved.

### Audit Event

An append-only record of significant actions.

Examples:
- Submission created.
- Document uploaded.
- Processing started.
- Processing failed.
- Validation completed.
- Reviewer assigned.
- Finding resolved.
- Configuration published.

Audit events record actor, action, target, timestamp, and relevant context.

## 3. Main relationships

- Institution has many submissions.
- Institution owns many document templates.
- Document template has many versions.
- Submission references one template version.
- Submission has one document object.
- Submission has many processing jobs over its lifecycle.
- Submission has many extraction runs.
- Extraction run has many field observations.
- Institution owns many reference-data sets.
- Reference-data set has many versions.
- Reference-data version has many canonical items and aliases.
- Institution owns many validation policies.
- Validation policy has many versions.
- Submission has many validation runs.
- Validation run references one policy version and the reference-data versions it used.
- Validation run produces many findings.
- Submission may have many reviews and audit events.

## 4. Outcome semantics

PASS:
All required checks completed successfully and no blocking or unresolved review conditions remain.

FAIL:
At least one configured deterministic check failed.

REVIEW:
The result is uncertain, ambiguous, incomplete, or requires human judgment.

A processing failure is not automatically a business FAIL. It is recorded as a processing error and handled according to retry and operational policy.

## 5. Important modeling distinction

Keep these concepts separate:

- Reference data: accepted classifications, codes, labels, and aliases.
- Master data: real-world customers, accounts, branches, and other operational records owned by the institution.
- Transactional data: submissions, processing attempts, reviews, and audit events.

Veriflow manages reference data and consumes authorized external validation sources. It does not become the institution's master-data platform.
