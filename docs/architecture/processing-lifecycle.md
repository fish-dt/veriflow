# Veriflow — Processing Lifecycle

## 1. Lifecycle overview

A submission moves through registration, upload, processing, validation, and optional human review.

Processing status and verification outcome are separate concepts.

## 2. Submission statuses

- DRAFT: submission registered but not ready for processing.
- AWAITING_UPLOAD: document has not yet been uploaded.
- READY: document is stored and ready for processing.
- PROCESSING: document engine is processing the document.
- VALIDATING: extraction observations are being evaluated.
- COMPLETED: automated verification has produced an outcome.
- REVIEW_REQUIRED: human review is required.
- REVIEWED: a reviewer has recorded a resolution.
- CANCELLED: submission was cancelled before completion.

The implementation must define allowed transitions and reject invalid transitions.

## 3. Processing job statuses

- PENDING
- PROCESSING
- COMPLETED
- FAILED
- CANCELLED

A job is distinct from a submission. A submission may have multiple jobs or attempts over time.

## 4. Processing sequence

### Stage 1 — Registration

- Authenticate the caller.
- Resolve the institution.
- Validate document type and template availability.
- Create the submission.
- Record external references where provided.

### Stage 2 — Upload

- Enforce configured file-size limits.
- Validate supported media types.
- Calculate a checksum.
- Store the original document in private MinIO storage.
- Record object metadata.
- Mark the submission READY.

### Stage 3 — Job creation

- Create a PENDING processing job.
- Record the applicable template version.
- Set attempt limits and availability.
- Ensure duplicate requests do not create duplicate work.

### Stage 4 — Document processing

The document engine:
- Claims an available job.
- Receives a time-limited lease.
- Retrieves the authorized document.
- Checks file integrity and supported format.
- Preprocesses pages.
- Performs OCR.
- Extracts configured fields.
- Produces observations with confidence and evidence.
- Reports completion or a classified error.

### Stage 5 — Validation

The verification API:
- Validates the returned observations.
- Applies required-field checks.
- Applies format and date constraints.
- Normalizes values.
- Matches values against the recorded reference-data versions.
- Applies policy rules.
- Creates findings.
- Produces PASS, FAIL, or REVIEW.

### Stage 6 — Human review

A reviewer:
- Opens the submission.
- Examines findings and supporting evidence.
- Records a resolution and reason.
- Adds comments where appropriate.
- Completes the review.

Review does not rewrite or erase the automated result.

### Stage 7 — Integration

When configured:
- The verification API records an outbound event.
- The integration gateway delivers the webhook.
- Delivery attempts and outcomes are recorded.
- Transient failures are retried according to policy.

## 5. Retry and lease requirements

- Jobs must have a bounded maximum attempt count.
- Workers must claim jobs atomically.
- A lease must expire if a worker disappears.
- Expired jobs must become eligible for recovery.
- Retry delays must be bounded and configurable.
- Permanent errors must not be retried indefinitely.
- Completion requests must be idempotent.
- A stale worker must not overwrite a newer successful attempt.

Use PostgreSQL transaction locking, including FOR UPDATE SKIP LOCKED, for safe job claiming.

## 6. Failure classification

Examples of retryable failures:
- Temporary object-storage outage.
- Temporary network failure.
- Temporary document-engine unavailability.

Examples of non-retryable failures:
- Unsupported file type.
- Corrupt or unreadable file.
- Invalid template configuration.
- File exceeds configured limits.

The system must preserve error codes and operational details without exposing internal stack traces to public clients.

## 7. Outcome rules

- Missing required information can produce FAIL.
- Ambiguous OCR or reference matching should produce REVIEW.
- An unreadable document must not produce PASS.
- A processing failure must not be represented as a business verification FAIL.
- An incomplete validation run must not produce PASS.
- Every outcome must be traceable to its evidence and configuration versions.

## 8. Audit requirements

Record:
- Submission creation and updates.
- Upload and checksum metadata.
- Job claims, attempts, and failures.
- Extraction run results.
- Policy and reference-data versions used.
- Validation outcomes and findings.
- Review assignments and resolutions.
- Webhook delivery attempts.

Audit history must not be silently overwritten by retries or subsequent reviews.
