# ADR-0002: Reference Data Management

Status: Accepted for MVP design

## Context

Document verification often requires comparing extracted values against institution-approved codes, classifications, and accepted labels.

Examples include:
- Country codes.
- Currency codes.
- Document types.
- Institution-defined branch codes.
- Customer categories.
- Product-specific classifications.

Hard-coding these values into application logic makes updates difficult and makes historical verification results difficult to reproduce.

## Decision

Veriflow will provide institution-scoped, versioned reference-data sets.

A reference-data set represents one governed domain of accepted values.

Each set contains published versions. A published version is immutable.

## Reference-data structure

A reference-data set contains:
- Stable set identifier.
- Institution identifier.
- Name and description.
- Domain or category.
- Ownership metadata.
- Current published version.

A reference-data version contains:
- Version identifier.
- Version number.
- Publication status.
- Source information.
- Import metadata.
- Effective date.
- Publication timestamp.

A reference-data item contains:
- Canonical code.
- Display label.
- Description.
- Status.
- Effective-from date.
- Effective-to date, optional.

An alias maps an alternate representation to a canonical item.

## Lifecycle

1. Administrator creates or selects a reference-data set.
2. Administrator imports or edits a draft version.
3. The system validates codes, duplicate entries, aliases, and effective dates.
4. Administrator reviews import errors.
5. Authorized administrator publishes the version.
6. Published version becomes immutable.
7. Future changes create a new version.

## Matching behavior

For extracted values:

1. Preserve the original extracted value.
2. Apply the configured normalization strategy.
3. Attempt an exact canonical-code match.
4. Attempt an exact alias match.
5. Apply explicitly configured deterministic normalization.
6. If no reliable match exists, create a finding.
7. If multiple plausible matches exist, require REVIEW.

Fuzzy matching must never silently convert an ambiguous value into an accepted canonical value.

## Historical reproducibility

Every validation run must record the exact reference-data version used.

Publishing a newer version must not change historical validation results.

Reprocessing must explicitly identify whether it uses:
- The original configuration snapshot; or
- A newly selected template, policy, and reference-data version.

## Data governance

Each set should record:
- Data owner.
- Source organization or system.
- Import method.
- Last verified date.
- Effective dates.
- Publication actor.
- Change history.

Institution administrators are responsible for confirming the authority and correctness of institution-specific data.

## Scope boundary

Reference data is not customer master data.

Veriflow may use authorized external master-data lookups through integrations, but it will not replicate or own the institution's full customer, account, or branch master.

## Consequences

Benefits:
- Explainable matching.
- Reproducible historical decisions.
- Institution-specific configuration.
- Controlled updates.
- Reduced hard-coded business logic.

Costs:
- Version lifecycle management.
- Import validation.
- Alias governance.
- Additional audit and storage requirements.
