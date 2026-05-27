# Adopt Type Checking Conditional Imports for Testing Dependencies: Modules Use String

These rules are ALWAYS ACTIVE for all Python modules that require type hints from testing or optional dependencies, including partner integration libraries and core langchain components that use optional dependencies for type hints.

### Rules

- **R-TYPECK-001** SHOULD: Modules SHOULD use string literals for type annotations when referencing types from TYPE_CHECKING imports in function signatures.

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
Claude Code MUST NOT skip or defer verification of TYPE_CHECKING usage. Pre-commit hooks and CI/CD pipelines MUST enforce this rule by scanning for unconditional imports from testing frameworks and running mypy/pyright type checkers on all Python modules.
</enforcement>