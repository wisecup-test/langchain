# Adopt pytest as Standard Unit Testing Framework: Test Functions Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains multiple partner integration libraries (openai, nomic) that require consistent testing approaches across different modules
- Unit tests are organized in dedicated test directories following the pattern libs/partners/{provider}/tests/unit_tests/ with specialized test files for different components
- Test files demonstrate a consistent pattern of testing framework usage across chat models, embeddings, imports, and standard interfaces
- The testing infrastructure needs to support both component-specific tests (client utilities, prompt caching) and standardized interface tests
- A unified testing framework is essential for maintaining code quality and reliability across the growing number of partner integrations

## Problem Statement

As the codebase scales with multiple partner integration libraries, inconsistent testing frameworks and practices can lead to fragmented test suites, reduced maintainability, and difficulty in establishing uniform quality standards. A standardized testing framework is needed to ensure consistent test structure, reliable execution, and efficient debugging across all partner libraries and components.

## Decision

1. MUST: Test functions MUST use pytest conventions (test_ prefix) for automatic discovery

## Policy Block

- MUST Test functions MUST use pytest conventions (test_ prefix) for automatic discovery

In scope:
- All unit tests in libs/partners/* directories
- Component-specific tests for chat models, embeddings, and utilities
- Standard interface validation tests
- Import verification tests
- Client utility tests

Out of scope:
- Integration tests that require external services
- End-to-end tests spanning multiple systems
- Performance benchmarking tests
- Manual testing procedures
- Documentation examples that are not executable tests

Exceptions:
- EX-001: Legacy tests exist in a different framework and migration cost exceeds benefit
- EX-002: Third-party library requires specific test runner for compatibility testing

## Rationale

- pytest is the de facto standard testing framework in the Python ecosystem with extensive community support, rich plugin ecosystem, and superior fixture management
- The detected pattern shows consistent pytest usage across 5 files with 91.08% confidence, indicating an established practice that should be formalized
- Standardizing on pytest enables consistent test execution, reporting, and debugging experiences across all partner integrations
- pytest's powerful features (fixtures, parametrization, markers) support both simple unit tests and complex testing scenarios without requiring different frameworks

## Consequences

Positive:
- Consistent testing experience across all partner libraries reduces cognitive load for developers
- pytest's extensive plugin ecosystem enables advanced testing capabilities (coverage, mocking, parallel execution) without framework changes
- Standardized test structure improves discoverability and maintainability of test suites
- Unified test execution simplifies CI/CD pipeline configuration and reduces build complexity
- Better error reporting and debugging capabilities through pytest's detailed assertion introspection

Negative:
- Teams familiar with other testing frameworks (unittest, nose) face a learning curve
- Existing tests in other frameworks require migration effort
- pytest dependency must be maintained and updated across all projects
- Some advanced pytest features may introduce complexity for simple test cases

## Alternatives

- Use Python's built-in unittest framework (rejected)
  Rejected because: unittest requires more boilerplate code, lacks advanced features like fixtures and parametrization, and provides less readable test output compared to pytest
  When valid: For projects with strict zero-dependency requirements or when integrating with Java-style testing patterns
- Allow each partner library to choose its own testing framework (rejected)
  Rejected because: Fragmented testing approaches increase maintenance burden, complicate CI/CD pipelines, and create inconsistent developer experiences across the codebase
  When valid: Never recommended for a unified codebase with shared quality standards
- Use nose2 as the testing framework (rejected)
  Rejected because: nose2 has limited active development and community support compared to pytest, and offers fewer modern testing features
  When valid: For legacy codebases already heavily invested in nose/nose2 where migration cost is prohibitive

## Risks

- pytest version incompatibilities across different partner libraries could cause test failures
  Mitigation: Pin pytest version in project dependencies and establish a coordinated upgrade process across all libraries
  Owner: Engineering team
- Over-reliance on pytest-specific features may make tests difficult to understand for developers unfamiliar with pytest
  Mitigation: Establish testing guidelines that balance pytest features with readability, provide pytest training resources, and document common patterns
  Owner: Engineering team
- Migration of existing non-pytest tests could introduce regressions or reduce test coverage temporarily
  Mitigation: Implement gradual migration strategy with parallel test execution during transition period and mandatory coverage verification
  Owner: Engineering team

## Implementation Notes

- Install pytest as a development dependency in each partner library's requirements or pyproject.toml
- Organize tests following the pattern: libs/partners/{provider}/tests/unit_tests/ with subdirectories for component types
- Create conftest.py files at appropriate levels to share fixtures across related tests
- Use pytest markers to categorize tests (e.g., @pytest.mark.unit, @pytest.mark.slow) for selective test execution
- Configure pytest.ini or pyproject.toml with project-specific pytest settings (test paths, markers, plugins)
- Leverage pytest-cov plugin for coverage reporting and pytest-xdist for parallel test execution in CI

## Continuation Context


Verify commands:
- grep -r "import pytest" libs/partners/*/tests/unit_tests/ | wc -l
- find libs/partners/*/tests/unit_tests -name 'test_*.py' -type f | head -5
- pytest --collect-only libs/partners/*/tests/unit_tests/ 2>&1 | grep -E '(test session starts|collected)'

Accept when:
- All test files in unit_tests directories follow the test_*.py naming convention
- pytest successfully discovers and collects tests from all partner library unit test directories
- No unittest.TestCase or nose-specific imports are present in unit test files
- CI pipeline executes tests using pytest command

## Enforcement

- Verified by: Automated CI checks that verify pytest is used for test execution
- Verified by: Code review process checks for pytest usage in new test files
- Verified by: Static analysis tools scan for non-pytest testing framework imports in unit test directories
- Verified by: Pre-commit hooks validate test file naming conventions
- Violation handling: CI pipeline fails if tests are not executable via pytest
- Violation handling: Code review blocks merge if non-pytest frameworks are introduced without documented exception
- Violation handling: Automated alerts notify team leads of testing framework violations
- Violation handling: Quarterly audits identify and prioritize migration of non-compliant tests
- Exception process: Submit exception request to engineering lead with justification and impact analysis
- Exception process: Architecture review board evaluates exceptions for third-party compatibility requirements
- Exception process: Approved exceptions must be documented in test directory README with migration timeline
- Exception process: All exceptions are reviewed annually for potential resolution