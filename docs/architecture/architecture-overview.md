# Veriflow — Architecture Overview

## 1. Architecture style

Veriflow uses three backend services and one web application.

The verification API is the system of record and owns business decisions. The document engine performs document processing and returns observations. The integration gateway handles outbound communication.

The operations console is a client application, not a fourth backend service.

## 2. Deployable components

### 2.1 Verification API

Location: apps/verification-api

Technology: Java 21, Spring Boot

Responsibilities:
- Authentication and authorization.
- Institution and user administration.
- Submission lifecycle.
- Document metadata and storage coordination.
- Document templates and template versions.
- Reference-data sets, versions, items, and aliases.
- Validation policies and policy versions.
- Processing-job creation and coordination.
- Observation ingestion.
- Business validation and outcome aggregation.
- Human review and resolution.
- Audit history.
- Webhook configuration and delivery records.

The verification API owns the authoritative business state.

### 2.2 Document Engine

Location: apps/document-engine

Technology: Python, FastAPI

Responsibilities:
- Retrieve authorized document content.
- Validate supported file formats.
- PDF and image preprocessing.
- OCR.
- Fixed-layout field extraction.
- Configured-region quality observations.
- Signature and stamp presence observations, where configured.
- Return structured observations with confidence and evidence.

The document engine does not:
- Own submission records.
- Access the business database directly.
- Manage reference data.
- Apply institution-specific business policies.
- Produce the final PASS, FAIL, or REVIEW outcome.

### 2.3 Integration Gateway

Location: apps/integration-gateway

Technology: Java 21, Spring Boot

Responsibilities:
- Deliver configured webhooks.
- Retry transient delivery failures.
- Record delivery attempts and outcomes through the verification API.
- Support outbound integration with external systems.

The verification API remains the source of truth for delivery configuration and state.

### 2.4 Operations Console

Location: apps/operations-console

Technology: Next.js, TypeScript

Responsibilities:
- Submission creation and inspection.
- Document upload.
- Processing status visibility.
- Findings and evidence presentation.
- Reviewer workflows.
- Configuration and reference-data administration, subject to permissions.

The console communicates through the public verification API. It does not connect directly to PostgreSQL or MinIO.

## 3. Shared infrastructure

### PostgreSQL

Stores:
- Institutions and users.
- Submissions and document metadata.
- Templates and versions.
- Reference data and versions.
- Policies and versions.
- Processing jobs.
- Extraction observations.
- Validation results and findings.
- Review records.
- Audit events.
- Webhook delivery state.

### MinIO

Stores private objects:
- Original uploaded documents.
- Derived processing artifacts when retention policy permits.
- Other document evidence that must be retained as files.

PostgreSQL stores object keys and metadata, not document binary content.

## 4. Processing communication

1. The client creates a submission through the verification API.
2. The API issues an upload authorization or accepts the file through a controlled upload endpoint.
3. The file is stored in MinIO.
4. The API records document metadata and creates a processing job.
5. The document engine claims a job through an authenticated internal API.
6. The document engine retrieves the authorized object and processes it.
7. The document engine submits structured observations to the verification API.
8. The API evaluates observations using the applicable template, policy, and reference-data versions.
9. The API records findings and the submission outcome.
10. The integration gateway delivers configured external events.

## 5. Job coordination

The verification API owns the PostgreSQL-backed job queue.

Internal operations:
- POST /internal/jobs/claim
- POST /internal/jobs/{jobId}/complete
- POST /internal/jobs/{jobId}/fail

Workers must use leases, bounded retries, and idempotent completion.

Workers do not receive direct database credentials.

## 6. Architectural principles

- Clear ownership of business state.
- Explicit service contracts.
- No shared database access by worker services.
- Versioned configuration.
- Evidence-backed outcomes.
- Idempotent processing.
- Least-privilege access.
- Private document storage.
- Human review for uncertainty.
- Avoid distributed infrastructure unless actual scale requires it.

## 7. Deployment model

Local development uses Docker Compose.

The initial deployment contains:
- Verification API.
- Document Engine.
- Integration Gateway.
- Operations Console.
- PostgreSQL.
- MinIO.

Production topology, high availability, backup, retention, and institution-specific security requirements must be designed separately before production deployment.
