# Standardize External API Integration with Explicit Import Guards: External Client Classes

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all external API integrations and third-party service wrappers within the codebase.

## Context

- The codebase integrates with numerous external APIs and third-party services (EdenAI, Weaviate, Yellowbrick, Bing Search, ClickUp, Brave Search, CometML, ClearML, Streamlit, etc.) that require optional dependencies
- External API clients may not be installed in all deployment environments, requiring graceful degradation when dependencies are missing
- Pattern detected across 28 files with 90% confidence indicates a consistent architectural approach to handling optional external dependencies
- Tools, callbacks, and vectorstore integrations represent the primary integration points with external services
- The pattern appears in both core langchain modules and partner library implementations, suggesting a cross-cutting architectural concern

## Problem Statement

How should the codebase handle integrations with external APIs and third-party services that have optional dependencies, ensuring that missing dependencies don't break the entire application while maintaining clear error messages and import-time validation?

## Decision

1. MUST: External API client classes MUST defer dependency imports until class instantiation or method invocation, not at module load time

## Policy Block

- MUST External API client classes MUST defer dependency imports until class instantiation or method invocation, not at module load time

In scope:
- All tool implementations that wrap external APIs
- All callback handlers that integrate with external monitoring or logging services
- All vectorstore implementations that connect to external database services
- Partner library integrations with third-party services
- Any module that imports optional dependencies for external service integration

Out of scope:
- Core framework dependencies that are always required
- Standard library imports
- Internal module imports within the same package
- Development and testing dependencies

Exceptions:
- EXC-001: A specific deployment environment guarantees the presence of all external dependencies
- EXC-002: The external API is being migrated from optional to required dependency

## Rationale

- Pattern detected in 28 files with 90% confidence indicates this is an established architectural practice across the codebase
- Lazy imports with explicit guards enable modular deployment where only needed external dependencies are installed, reducing deployment size and complexity
- Clear error messages at import time improve developer experience by immediately identifying missing dependencies rather than cryptic runtime errors
- This pattern supports the plugin architecture where external integrations are optional extensions rather than core requirements

## Consequences

Positive:
- Reduced deployment footprint as only required external dependencies need to be installed
- Improved error messages guide developers to install missing packages with clear instructions
- Better modularity allowing users to choose which external services to integrate
- Graceful degradation when optional services are unavailable

Negative:
- Increased code complexity with try-except blocks and import guards throughout integration modules
- Potential for runtime errors if import guards are not properly implemented or tested
- More difficult to statically analyze dependencies and ensure type safety
- Additional testing burden to verify behavior with and without optional dependencies

## Alternatives

- Make all external API dependencies required and install them by default (rejected)
  Rejected because: Would significantly increase deployment size and force users to install dependencies for services they don't use, violating the principle of minimal dependencies
  When valid: In specialized distributions or Docker images targeting specific use cases where all integrations are needed
- Use separate packages for each external integration (e.g., langchain-weaviate, langchain-edenai) (accepted)
  When valid: This is the preferred approach for partner integrations as evidenced by libs/partners structure, but import guards are still needed within each package
- Fail fast at application startup if any external dependency is missing (rejected)
  Rejected because: Would prevent the application from running even when the missing dependency is for an unused feature, reducing flexibility
  When valid: In production environments with strict dependency management where all required services must be available

## Risks

- Import guards may be forgotten in new external API integrations, causing hard failures
  Mitigation: Implement linting rules and code review checklists to verify import guards in all external integration modules
  Owner: Engineering team
- Inconsistent error messages across different integrations may confuse users
  Mitigation: Create a standard error message template and helper function for missing dependency errors
  Owner: Developer experience team
- Testing coverage may miss scenarios where dependencies are missing
  Mitigation: Add CI test matrix that runs tests both with and without optional dependencies installed
  Owner: QA and CI/CD team

## Implementation Notes

- Use a consistent pattern: wrap external imports in try-except blocks at the point of use (class methods or __init__), not at module level
- Include the package name and installation command in ImportError messages: 'Could not import {package}. Please install it with `pip install {package}`'
- Consider creating a utility function like `require_optional_dependency(package_name, import_name)` to standardize import guard implementation
- Document optional dependencies in setup.py/pyproject.toml extras_require section and reference them in error messages
- For type checking, use TYPE_CHECKING imports or string annotations to avoid runtime import requirements

## Continuation Context


Verify commands:
- grep -r 'import.*try:' --include='*.py' libs/langchain/langchain_classic/tools/ libs/langchain/langchain_classic/callbacks/ libs/langchain/langchain_classic/vectorstores/
- grep -r 'ImportError.*install' --include='*.py' libs/
- python -c "import ast; import sys; [print(f.name) for f in ast.walk(ast.parse(open(sys.argv[1]).read())) if isinstance(f, ast.Try)]"

Accept when:
- All external API integration modules contain try-except blocks around optional dependency imports
- ImportError messages include package names and installation instructions
- Modules can be imported successfully even when external dependencies are not installed
- CI tests pass both with and without optional dependencies in the environment

## Enforcement

- Verified by: Automated linting rules checking for import patterns in external integration modules
- Verified by: Code review checklist requiring verification of import guards for new external API integrations
- Verified by: CI test matrix running tests with minimal dependencies and with full optional dependencies
- Violation handling: Linter failures block PR merge until import guards are added
- Violation handling: Code review requires explicit acknowledgment of import guard verification
- Violation handling: Failed CI tests with missing dependencies trigger automatic review request
- Exception process: Request exception through architecture review board with justification
- Exception process: Document the exception reason in code comments with reference to approval
- Exception process: Add to technical debt backlog for future remediation if temporary exception