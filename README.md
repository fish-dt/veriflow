# Veriflow

Document verification infrastructure for financial institutions.

## Frozen MVP decisions

- Document layout: Fixed
- Submission: One document per submission
- Architecture: Three services
- Object storage: MinIO
- Core backend: Java 21 + Spring Boot modular monolith
- Document intelligence: Python + FastAPI
- Integration: Spring Boot
- Database: PostgreSQL
- Job queue: PostgreSQL-backed
- Deployment: Docker Compose

## Services

1. Core API
2. Document Intelligence
3. Integration Service

## MVP workflow

1. Create a document submission.
2. Upload one document.
3. Store the original document in private object storage.
4. Create a processing job.
5. Extract configured fields and document observations.
6. Validate extracted information against configured rules and reference data.
7. Return PASS, FAIL, or REVIEW.
8. Support human review and maintain an audit trail.

## Principles

- API-first
- Fixed-layout document processing
- Human oversight for uncertain results
- Versioned configuration and reference data
- Evidence-based validation
- Open-source development stack
