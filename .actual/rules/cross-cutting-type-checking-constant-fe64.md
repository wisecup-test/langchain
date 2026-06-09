# Adopt Type Checking Conditional Imports for Testing Dependencies: Type Checking Constant

These rules are ALWAYS ACTIVE for all Python modules that require type hints from testing or optional dependencies, including all modules in partner integration libraries (libs/partners/*), core langchain modules using optional dependencies for type hints, and any module importing from pytest, unittest, or other testing frameworks solely for type annotations.

### Rules

- **R-TYPE-001** MUST: Import the TYPE_CHECKING constant from the typing module at the top of files using conditional type imports.
- **R-TYPE-002** MUST: Place TYPE_CHECKING blocks after standard library imports but before the main code.
- **R-TYPE-003** MUST: Use string literals when referencing TYPE_CHECKING imports in function signatures to avoid runtime NameError exceptions.
- **R-TYPE-004** MUST: Group related TYPE_CHECKING imports together and add comments explaining why they are conditional.
- **R-TYPE-005** SHOULD: Run mypy or pyright in CI to ensure type checking still works with conditional imports.

### Verify

```bash
# Count TYPE_CHECKING imports in partner libraries
grep -r 'from typing import TYPE_CHECKING' libs/partners/*/langchain_*/*.py | wc -l

# Count TYPE_CHECKING conditional blocks in codebase (excluding tests)
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
Claude Code MUST NOT skip or defer verification. Pre-commit hooks MUST block commits with unconditional testing imports. CI/CD builds MUST fail if type checking does not pass or if testing dependencies appear in production requirements.
</enforcement>