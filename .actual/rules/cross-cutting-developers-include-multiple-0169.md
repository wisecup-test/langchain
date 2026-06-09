# Adopt Type Checking Conditional Imports for Testing Dependencies: Developers Include Multiple

These rules are ALWAYS ACTIVE for all Python modules that require type hints from testing or optional dependencies, including partner integration libraries and core langchain components.

### Rules

- **R-TYPECK-001** MAY: Developers MAY include multiple conditional import blocks for different categories of optional dependencies (testing, development tools, optional integrations).
- **R-TYPECK-002** MUST: Always import TYPE_CHECKING at the top of the file: `from typing import TYPE_CHECKING`.
- **R-TYPECK-003** MUST: Place TYPE_CHECKING blocks after standard library imports but before the main code.
- **R-TYPECK-004** MUST: When using TYPE_CHECKING imports in function signatures, use string literals for type annotations.
- **R-TYPECK-005** SHOULD: Group related TYPE_CHECKING imports together and add comments explaining why they're conditional.
- **R-TYPECK-006** MUST NOT: Include testing framework imports (pytest, unittest) or optional development dependencies unconditionally in runtime code.
- **R-TYPECK-007** MUST NOT: Use TYPE_CHECKING imports directly in runtime code without string annotations, as this will cause NameError exceptions in production.

### Verify

```bash
# Count TYPE_CHECKING imports in partner libraries
grep -r 'from typing import TYPE_CHECKING' libs/partners/*/langchain_*/*.py | wc -l

# Count conditional TYPE_CHECKING blocks in codebase (excluding tests)
grep -r 'if TYPE_CHECKING:' libs/ --include='*.py' | grep -v test | wc -l

# Run mypy type checking on modules using TYPE_CHECKING imports
python -m mypy libs/partners/ollama/langchain_ollama/ --strict --no-error-summary 2>&1 | grep -c 'Success'

# Verify no unconditional testing framework imports in non-test files
grep -r '^from pytest import\|^from unittest import\|^import pytest\|^import unittest' libs/ --include='*.py' | grep -v 'tests/' | wc -l
```

**Accept when:**
- All Python modules with optional type dependencies use TYPE_CHECKING conditional imports
- Static type checking (mypy/pyright) passes successfully on all modules using TYPE_CHECKING imports
- Production installations do not require testing frameworks or development-only dependencies
- No NameError exceptions occur in production due to missing TYPE_CHECKING imports
- Unconditional testing framework imports are not found in non-test files

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks MUST block commits with unconditional testing imports. CI/CD pipeline MUST run mypy and pyright type checkers on all Python modules. Code review MUST verify TYPE_CHECKING usage for optional dependencies.
</enforcement>