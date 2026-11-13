# Architecture Decision Record

## ADR-001: Veriflow MVP Architecture

Status: Accepted

### 1. Document model
- Fixed-layout documents.
- One document per submission.
- Configured templates define expected fields and validation regions.

### 2. Service architecture

Core API:
- Authentication and authorization.
- Submission lifecycle.
- Document metadata.
- Template and validation configuration.
- Reference data.
- Job creation and lifecycle.
- Result aggregation.
- Human review and audit history.

Document Intelligence:
- PDF and image processing.
- OCR.
- Configured field extraction.
- Document quality observations.
- Signature and stamp presence detection.
- Structured results with confidence and evidence.

Integration Service:
- Webhook delivery.
- Retry handling.
- Delivery status tracking.
- External system notifications.

### 3. Data infrastructure
- PostgreSQL: business data, metadata, jobs and results.
- MinIO: private original and processed document storage.
- PostgreSQL-backed job queue.
- REST communication between services.

### 4. Processing ownership
Core API owns business decisions.
Document Intelligence returns observations, not approval decisions.
Integration Service delivers events and notifications.

### 5. Decision outcomes
- PASS
- FAIL
- REVIEW

Uncertain extraction or validation must not silently become PASS.

### 6. Explicit MVP exclusions
- Variable-layout document understanding.
- Multiple documents in one submission.
- Automated banking approval.
- Biometric identity verification.
- Sanctions and PEP screening.
- Signature authenticity verification.
- Kafka, RabbitMQ and Redis.

### 7. Deployment
Docker Compose for local development and pilot deployment.
