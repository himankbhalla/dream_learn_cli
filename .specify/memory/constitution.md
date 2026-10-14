<!--
Sync Impact Report
Version change: uninitialized scaffold to 1.0.0
Modified principles:
- Scaffold principle 1 placeholder to I. Strict TypeScript
- Scaffold principle 2 placeholder to II. Test-First
- Scaffold principle 3 placeholder to III. Validated API Boundaries
- Scaffold principle 4 placeholder to IV. Security-First
- Scaffold principle 5 placeholder to V. Twelve-Factor Configuration
- Added VI. Accessible Interfaces
- Added VII. Small Modules and Library-First Design
Added sections: Additional Constraints; Development Workflow
Removed sections: None
Follow-up TODOs: Confirm the original ratification date.
-->
# Dream Learn CLI Constitution

## Core Principles

### I. Strict TypeScript
All TypeScript projects MUST enable strict mode. Source code MUST NOT use `any`, including
explicit annotations and suppressions that bypass type checking. Types MUST describe domain
and API contracts accurately so errors are caught during development rather than at runtime.

### II. Test-First
Behavioral changes MUST begin with tests that demonstrate the expected behavior and fail before
implementation. Implementations MUST pass those tests, and refactoring MUST preserve the tested
behavior. Tests MUST cover relevant success paths, failure paths, and boundary conditions.

### III. Validated API Boundaries
Data entering through an API boundary MUST be validated with Zod before application logic trusts
or uses it. This includes inbound requests and responses from external services. Validation
failures MUST produce explicit, appropriately handled errors and MUST NOT be converted into
success-shaped defaults.

### IV. Security-First
Changes MUST be assessed against applicable OWASP Top 10 risks. Secrets MUST NOT be committed
to source code or exposed in logs, errors, or client-visible output. Security-sensitive inputs
MUST be validated, and access MUST be limited to the minimum required.

### V. Twelve-Factor Configuration
Deploy-specific configuration MUST be supplied through the environment rather than embedded in
source code. Applications MUST validate required configuration at startup and report missing
or invalid values clearly without exposing secrets.

### VI. Accessible Interfaces
User interfaces MUST use semantic HTML and meet WCAG 2.1 AA accessibility requirements.
Interactive controls MUST be keyboard-operable and expose accessible names and state.

### VII. Small Modules and Library-First Design
Modules MUST have focused responsibilities and explicit interfaces. Reusable functionality
SHOULD be designed as independently testable libraries before being coupled to a particular
interface or runtime. Abstractions MUST be justified by reuse or clear separation of concerns.

## Additional Constraints

These principles apply to production code, tests, and supporting configuration where relevant.
TypeScript compiler options MUST preserve strict checking, and validation or accessibility
requirements MUST NOT be bypassed solely to simplify implementation.

## Development Workflow

Changes MUST include tests appropriate to their behavior and pass the repository's available
type-checking, linting, and test checks before review. Reviewers MUST verify compliance with
the principles above, including API validation, security, configuration handling, and
accessibility for user-facing changes.

## Governance

This constitution governs engineering decisions and takes precedence over conflicting local
conventions. Amendments MUST be reviewed, documented in the constitution, and reflected in its
version and amendment date. Changes that add or materially expand a principle or section require
a MINOR version increment; incompatible removals or redefinitions require a MAJOR increment;
clarifications and non-semantic edits require a PATCH increment. Compliance MUST be checked
during implementation and code review. Exceptions MUST be documented with their rationale and
approved during review.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE): original adoption date unknown | **Last Amended**: 2026-10-13
