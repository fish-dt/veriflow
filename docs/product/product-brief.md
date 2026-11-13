# Veriflow — Product Brief

## 1. Product definition

Veriflow is document verification infrastructure for financial institutions and fintech companies.

It receives structured document submissions, extracts configured information, evaluates that information against institution-defined policies and reference data, and returns an explainable verification outcome.

Veriflow is designed to integrate with existing onboarding platforms and operational systems. It does not replace core banking, customer management, or identity-provider systems.

## 2. Problem

Financial institutions receive documents through digital onboarding and other customer-facing workflows.

Operational teams may need to:
- Check whether required fields are present.
- Verify that documents follow an expected layout.
- Compare extracted values with accepted reference values.
- Identify inconsistent or unreadable information.
- Review exceptions and retain evidence of decisions.

Manual checks can be slow and inconsistent. Poorly explained automation can create additional operational and compliance risk.

## 3. Proposed solution

Veriflow provides a configurable document verification workflow:

1. An authorized system or operator creates a submission.
2. A document is uploaded and securely stored.
3. The document engine processes the file using a configured template.
4. Extracted observations are returned with confidence and evidence.
5. The verification API evaluates observations against policies and reference data.
6. Veriflow produces PASS, FAIL, or REVIEW.
7. Authorized reviewers can inspect findings and record a resolution.
8. The system preserves processing history and audit evidence.

## 4. Target customers

Initial target segments:
- Fintech companies.
- Microfinance institutions.
- Digital banking providers.
- Financial institutions with digital onboarding workflows.
- Technology providers integrating onboarding services.

The initial product should be configurable for different institutions rather than hard-coded to one bank or regulatory environment.

## 5. Primary users

### Integration engineer
Integrates Veriflow with an existing onboarding or application platform.

### Operations analyst
Monitors submissions and investigates verification findings.

### Reviewer
Reviews uncertain or failed checks, examines evidence, and records a resolution.

### Configuration administrator
Maintains document templates, reference data, policies, and access permissions.

### Auditor or compliance observer
Inspects historical decisions, evidence, configuration versions, and audit events.

## 6. Core product capabilities

- Document submission and secure storage.
- Fixed-layout document templates.
- OCR and configured field extraction.
- Field-level confidence and evidence.
- Document quality observations.
- Versioned reference data.
- Configurable validation policies.
- Cross-field consistency checks within a submission.
- Explainable PASS, FAIL, and REVIEW outcomes.
- Human review and resolution.
- Audit history.
- API integration and webhook delivery.

## 7. Product boundaries

Veriflow does not independently:
- Establish a person's legal identity.
- Authenticate signatures, stamps, or seals.
- Perform biometric identity matching.
- Perform sanctions or PEP screening.
- Make credit decisions.
- Approve or reject bank accounts.
- Replace a financial institution's customer master or core banking system.

Signature and stamp checks, if enabled, indicate presence in configured regions only. They do not establish authenticity.

Veriflow provides verification evidence and workflow outcomes. The institution retains responsibility for consequential business decisions.

## 8. MVP constraints

- Fixed-layout documents.
- One document per submission.
- Three backend services.
- One operations console.
- PostgreSQL for transactional data and job coordination.
- MinIO for private document storage.
- No Kafka, RabbitMQ, or Redis.
- Docker Compose for local development.

## 9. MVP success criteria

A demonstrable MVP must allow an authorized user or integrated system to:

1. Register a document submission.
2. Upload one document.
3. Select an applicable template.
4. Process the document.
5. Extract configured fields.
6. Evaluate required fields and configured validation rules.
7. Normalize supported values against versioned reference data.
8. Return an explainable outcome.
9. Review findings and record a resolution.
10. Inspect the submission's processing and audit history.

## 10. Product validation

Before treating the product as commercially validated, interview target financial-institution operations, compliance, and technology teams.

Validate:
- Which document types cause the most manual work.
- How document requirements are maintained.
- Which reference data sources are authoritative.
- What evidence reviewers need.
- Which existing systems must integrate.
- What deployment and data-residency requirements apply.
- Who owns final approval decisions.
