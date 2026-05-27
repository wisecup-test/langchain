# Standardize Public API Contract Testing with Unit Test Validation: Public External Implementations

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all public/external API implementations and partner integrations within the codebase.

## Context

- The codebase contains multiple partner integrations (Ollama, OpenAI, Nomic) that expose public APIs requiring consistent contract validation
- Unit tests across partner libraries demonstrate a pattern of standardized testing for embeddings, chat models, and client utilities
- Public APIs require rigorous contract testing to ensure backward compatibility and prevent breaking changes for external consumers
- The pattern appears in 9 files with 91.93% confidence, indicating a well-established practice across the partner integration ecosystem
- Base classes and memory abstractions require stable public interfaces that are validated through comprehensive unit testing

## Problem Statement

Without standardized contract testing for public and external APIs, partner integrations risk introducing breaking changes, inconsistent behavior across implementations, and unreliable interfaces for downstream consumers. The lack of systematic validation makes it difficult to maintain API stability and detect regressions early in the development cycle.

## Decision

1. MUST: All public/external API implementations MUST include comprehensive unit tests that validate the API contract

## Policy Block

- MUST All public/external API implementations MUST include comprehensive unit tests that validate the API contract

In scope:
- All partner integration libraries under libs/partners/*
- Public API classes in langchain_classic (chains, memory, base classes)
- Chat models, embeddings, and client utility modules exposed to external consumers
- Router implementations and base memory abstractions with public interfaces
- Any module or class intended for direct consumption by external developers

Out of scope:
- Internal utility functions not exposed in public APIs
- Private implementation details marked with underscore prefixes
- Experimental features not yet stabilized for public use
- Development and debugging tools not intended for production use

Exceptions:
- EXC-001: Legacy APIs in deprecation phase with documented sunset timelines
- EXC-002: Prototype or alpha-stage features explicitly marked as unstable

## Rationale

- Pattern detected across 9 files with 91.93% confidence indicates this is an established best practice in the codebase
- Unit test validation provides early detection of contract violations before they reach production or external consumers
- Standardized testing across partner integrations ensures consistent quality and reliability across different API providers
- Contract testing reduces the risk of breaking changes and improves maintainability of public interfaces over time

## Consequences

Positive:
- Improved API stability and backward compatibility through systematic contract validation
- Early detection of breaking changes during development rather than in production
- Consistent quality standards across all partner integrations and public APIs
- Reduced maintenance burden through automated regression detection
- Increased confidence for external consumers relying on stable public interfaces

Negative:
- Additional development time required to write and maintain comprehensive unit tests
- Increased test suite execution time as contract tests are added for all public APIs
- Potential for test maintenance overhead when APIs evolve legitimately
- May slow down rapid prototyping of new experimental features

## Alternatives

- Rely solely on integration tests without dedicated unit-level contract tests (rejected)
  Rejected because: Integration tests are slower, more brittle, and provide less precise feedback about contract violations. Unit tests enable faster iteration and clearer failure diagnostics.
  When valid: May be acceptable for internal APIs with limited external exposure
- Use runtime contract validation with schema libraries instead of tests (rejected)
  Rejected because: Runtime validation catches issues too late (in production) and doesn't verify behavioral contracts, only data shapes. Tests provide comprehensive validation during development.
  When valid: Can be used as a complementary defense-in-depth measure alongside unit tests
- Manual code review as primary contract validation mechanism (rejected)
  Rejected because: Manual review is error-prone, doesn't scale, and provides no automated regression detection. Human review should complement, not replace, automated testing.
  When valid: Appropriate as an additional quality gate but insufficient as sole validation

## Risks

- Test coverage gaps may leave critical contract violations undetected
  Mitigation: Implement code coverage tracking with minimum thresholds (e.g., 80%) for public API modules. Use coverage reports in CI to identify gaps.
  Owner: Engineering team and QA
- Tests may become outdated as APIs evolve, creating false confidence
  Mitigation: Establish regular test review cycles. Require test updates as part of any API change PR. Use mutation testing to verify test effectiveness.
  Owner: Development teams and code reviewers
- Overly rigid contract tests may impede legitimate API evolution
  Mitigation: Design tests to validate essential contracts while allowing implementation flexibility. Use semantic versioning and deprecation periods for breaking changes.
  Owner: Architecture team and API maintainers

## Implementation Notes

- Start by identifying all public API surfaces in partner libraries and core modules. Create an inventory of modules requiring contract tests.
- Follow the established pattern: create 'tests/unit_tests' directories with 'test_standard.py' for core contracts and 'test_imports.py' for public interface validation.
- For each public class or function, write tests covering: valid inputs with expected outputs, invalid inputs with expected errors, edge cases, and backward compatibility scenarios.
- Integrate contract tests into CI pipeline with mandatory pass requirements. Configure test runners to execute unit tests before integration tests for fast feedback.
- Document the public API contract in docstrings and maintain a changelog tracking API modifications to help test authors understand what needs validation.

## Continuation Context


Verify commands:
- find libs/partners/*/tests/unit_tests -name 'test_*.py' -type f | wc -l
- grep -r 'def test_' libs/partners/*/tests/unit_tests/ | wc -l
- pytest libs/partners/ -v --tb=short -k 'test_standard or test_imports'

Accept when:
- Each partner integration library contains a 'tests/unit_tests' directory with at least 'test_standard.py' and 'test_imports.py' files
- All public API classes have corresponding unit tests with minimum 80% code coverage
- CI pipeline executes unit tests successfully with zero failures before allowing merge to main branch

## Enforcement

- Verified by: Automated CI pipeline checks that execute unit tests on every pull request
- Verified by: Code coverage reports generated during CI with minimum threshold enforcement
- Verified by: Mandatory code review checklist item verifying unit tests exist for any public API changes
- Violation handling: Pull requests without required unit tests for public API changes are blocked from merging
- Violation handling: CI pipeline fails if unit tests do not pass or coverage falls below threshold
- Violation handling: Automated notifications sent to PR author and reviewers when contract test requirements are not met
- Exception process: Request exception through architecture review board with written justification
- Exception process: Document exception rationale in ADR or technical debt tracking system
- Exception process: Set timeline for remediation if exception is temporary, with follow-up tasks created