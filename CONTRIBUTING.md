# Contributing to Veriflow

Thank you for your interest in contributing.

## Before contributing

- Read the project README and architecture documentation.
- Check existing issues before starting substantial work.
- Open an issue to discuss significant changes before implementation.

## Development principles

- Preserve clear service boundaries.
- Keep business decisions in the Verification API.
- Keep document extraction separate from verification decisions.
- Preserve auditability and historical reproducibility.
- Never commit credentials, customer data, or real identity documents.
- Include tests for behavioral changes.

## Pull requests

Every pull request should:

1. Explain the problem and proposed solution.
2. Identify affected services and contracts.
3. Include relevant tests.
4. Update documentation when behavior or architecture changes.
5. Describe security or data-handling implications.

## Commit messages

Use Conventional Commits:

- feat: add functionality
- fix: correct behavior
- docs: documentation changes
- test: add or update tests
- refactor: restructure without behavior changes
- chore: maintenance
- ci: pipeline changes
