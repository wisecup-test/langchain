# Adopt Type Checking Conditional Imports for Testing Dependencies: Type Checking Blocks

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all Python modules that require type hints from testing or optional dependencies.

## Context

- Python type checking tools like mypy require imports to resolve type annotations, but runtime code should not fail if optional testing dependencies are unavailable
- The codebase contains multiple partner integrations (Ollama, Nomic) and core components that need type hints for development but must remain functional without test dependencies installed
- Pattern detected across 4 files with 91.60% confidence indicates a consistent architectural approach to handling TYPE_CHECKING imports
- The pattern appears in both partner libraries and core langchain components, suggesting it is a cross-cutting concern for the entire ecosystem
- Development environments need rich type information while production deployments should minimize dependencies and avoid import errors from missing test packages

## Problem Statement

How can we provide comprehensive type hints for development and static analysis without creating runtime dependencies on testing frameworks or optional packages that may not be installed in production environments?

## Decision

1. SHOULD: TYPE_CHECKING blocks SHOULD be placed immediately after standard library imports and before third-party imports

## Policy Block

- SHOULD TYPE_CHECKING blocks SHOULD be placed immediately after standard library imports and before third-party imports

In scope:
- All Python modules in partner integration libraries (libs/partners/*)
- Core langchain modules that use optional dependencies for type hints
- Any module that imports from pytest, unittest, or other testing frameworks solely for type annotations
- Modules importing from optional third-party libraries used only in development

Out of scope:
- Test files themselves (tests/**/*.py) where testing dependencies are expected to be present
- Imports that are required at runtime for actual functionality, not just type hints
- Standard library imports that are always available
- Core dependencies listed in the package's required dependencies

Exceptions:
- EXC-001: A testing dependency is actually required at runtime for core functionality (not just type hints)
- EXC-002: Using Python 3.7+ postponed evaluation of annotations (from __future__ import annotations) making all annotations strings by default

## Rationale

- The pattern appears in 4 files across different components (Ollama chat models, Ollama LLMs, query constructors, and Nomic embeddings) with 91.60% confidence, indicating a deliberate and consistent architectural choice
- TYPE_CHECKING is a Python typing module constant that is True during static type checking but False at runtime, enabling zero-cost type hints
- This approach allows rich IDE support and mypy validation during development without imposing dependency requirements on production deployments
- Separating type-time imports from runtime imports reduces package size, installation complexity, and potential security surface area in production

## Consequences

Positive:
- Production deployments avoid unnecessary dependencies on testing frameworks and development tools
- Developers get full type checking and IDE autocomplete support during development
- Package installation is faster and lighter with fewer transitive dependencies
- Static type checkers like mypy can validate code correctness without runtime overhead
- Clear separation between development-time and runtime dependencies improves dependency management

Negative:
- Developers must remember to use string annotations or forward references when using TYPE_CHECKING imports in runtime code
- Slightly more verbose import sections with conditional blocks
- Potential for runtime errors if TYPE_CHECKING imports are accidentally used directly without proper string annotations
- Requires understanding of Python's type checking mechanics and the TYPE_CHECKING constant

## Alternatives

- Import all type dependencies unconditionally and add them to package requirements (rejected)
  Rejected because: Would bloat production dependencies with testing frameworks and development tools, increasing installation size and complexity unnecessarily
  When valid: Only valid for packages where testing dependencies are actually required at runtime
- Use try-except blocks around imports to handle missing optional dependencies (rejected)
  Rejected because: Try-except imports execute at runtime and add overhead; TYPE_CHECKING is evaluated at parse time with zero runtime cost
  When valid: Appropriate for truly optional runtime features, not for type-only imports
- Use from __future__ import annotations to make all annotations strings by default (deferred)
  Rejected because: While this enables forward references automatically, it requires Python 3.7+ and changes annotation semantics globally; TYPE_CHECKING is more explicit and granular
  When valid: Can be used in conjunction with TYPE_CHECKING for additional flexibility in modern Python codebases

## Risks

- Developers may accidentally use TYPE_CHECKING imports directly in runtime code, causing NameError exceptions in production
  Mitigation: Add linting rules and runtime tests that verify imports are available; use string annotations for TYPE_CHECKING types in signatures
  Owner: Engineering team and CI/CD pipeline
- Inconsistent application of the pattern across the codebase could lead to confusion and maintenance burden
  Mitigation: Document the pattern in contribution guidelines; add pre-commit hooks to detect unconditional testing imports; conduct code review training
  Owner: Architecture team and code reviewers
- Type checkers may behave differently than runtime, creating false confidence in type safety
  Mitigation: Maintain comprehensive test coverage including integration tests; use runtime type checking libraries like pydantic for critical paths
  Owner: QA team and test automation

## Implementation Notes

- Always import TYPE_CHECKING at the top: `from typing import TYPE_CHECKING`
- Place TYPE_CHECKING blocks after standard library imports but before the main code: `if TYPE_CHECKING:\n    from pytest import fixture`
- When using TYPE_CHECKING imports in function signatures, use string literals: `def foo(param: 'OptionalType') -> None:`
- For Python 3.10+, consider using union syntax with strings: `def foo(param: 'Type1 | Type2') -> None:`
- Group related TYPE_CHECKING imports together and add comments explaining why they're conditional
- Run mypy or pyright in CI to ensure type checking still works with conditional imports

## Continuation Context


Verify commands:
- grep -r 'from typing import TYPE_CHECKING' libs/partners/*/langchain_*/*.py | wc -l
- grep -r 'if TYPE_CHECKING:' libs/ --include='*.py' | grep -v test | wc -l
- python -m mypy libs/partners/ollama/langchain_ollama/ --strict --no-error-summary 2>&1 | grep -c 'Success'

Accept when:
- All Python modules with optional type dependencies use TYPE_CHECKING conditional imports
- Static type checking (mypy/pyright) passes successfully on all modules using TYPE_CHECKING imports
- Production installations do not require testing frameworks or development-only dependencies
- No NameError exceptions occur in production due to missing TYPE_CHECKING imports

## Enforcement

- Verified by: Pre-commit hooks scanning for unconditional imports from testing frameworks
- Verified by: CI/CD pipeline running mypy and pyright type checkers on all Python modules
- Verified by: Code review checklist requiring verification of TYPE_CHECKING usage for optional dependencies
- Verified by: Automated dependency analysis ensuring test frameworks are not in production requirements
- Violation handling: Pre-commit hooks block commits with unconditional testing imports
- Violation handling: CI builds fail if type checking does not pass or if testing dependencies appear in production requirements
- Violation handling: Code review process flags and requests changes for violations
- Violation handling: Runtime monitoring alerts on NameError exceptions related to missing type imports
- Exception process: Developer submits exception request with justification for why testing dependency is needed at runtime
- Exception process: Architecture team reviews whether the dependency truly needs to be runtime vs. type-only
- Exception process: If approved, dependency must be added to package requirements and documented in ADR exceptions
- Exception process: Exception is logged and reviewed quarterly to determine if it can be refactored away