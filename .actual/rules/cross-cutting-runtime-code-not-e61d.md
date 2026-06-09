# Adopt Type Checking Conditional Imports for Testing Dependencies: Runtime Code Not

These rules are ALWAYS ACTIVE for all Python modules that require type hints from testing or optional dependencies, including all modules in partner integration libraries (libs/partners/*), core langchain modules using optional dependencies for type hints, and any module importing from pytest, unittest, or other testing frameworks solely for type annotations.

### Rules

- **R-TYPECK-001** MUST NOT: Runtime code MUST NOT directly reference types imported under TYPE_CHECKING blocks without string annotations or forward references.

### Verify

```bash
# Count TYPE_CHECKING imports in partner libraries
grep -r 'from typing import TYPE_CHECKING' libs/partners/*/langchain_*/*.py | wc -l

# Count TYPE_CHECKING conditional blocks in non-test code
grep -r 'if TYPE_CHECKING:' libs/ --include='*.py' | grep -v test | wc -l

# Verify mypy type checking passes
python -m mypy libs/partners/ollama/langchain_ollama/ --strict --no-error-summary 2>&1 | grep -c 'Success'
```

**Accept when:**
- All Python modules with optional type dependencies use TYPE_CHECKING conditional imports
- Static type checking (mypy/pyright) passes successfully on all modules using TYPE_CHECKING imports
- Production installations do not require testing frameworks or development-only dependencies
- No NameError exceptions occur in production due to missing TYPE_CHECKING imports

<enforcement>
Claude Code MUST NOT skip or defer verification of TYPE_CHECKING usage. Pre-commit hooks and CI/CD pipelines MUST enforce this rule by scanning for unconditional imports from testing frameworks, running mypy/pyright type checkers, and analyzing dependencies to ensure test frameworks are not in production requirements.
</enforcement>