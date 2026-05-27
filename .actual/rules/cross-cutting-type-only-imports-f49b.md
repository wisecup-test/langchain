# Adopt Type Checking Conditional Imports for Testing Dependencies: Type Only Imports

These rules are ALWAYS ACTIVE for all Python modules that require type hints from testing or optional dependencies, including all partner integration libraries and core langchain components.

### Rules

- **R-TYPE-001** MUST: All type-only imports from testing frameworks or optional dependencies MUST be placed within a `if TYPE_CHECKING:` conditional block.
- **R-TYPE-002** MUST: Always import TYPE_CHECKING at the top of the module: `from typing import TYPE_CHECKING`.
- **R-TYPE-003** MUST: Place TYPE_CHECKING blocks after standard library imports but before the main code.
- **R-TYPE-004** MUST: When using TYPE_CHECKING imports in function signatures, use string literals for type annotations.
- **R-TYPE-005** SHOULD: Group related TYPE_CHECKING imports together and add comments explaining why they're conditional.
- **R-TYPE-006** SHOULD: For Python 3.10+, consider using union syntax with strings in annotations.

### Verify

```bash
# Count TYPE_CHECKING imports in partner libraries
grep -r 'from typing import TYPE_CHECKING' libs/partners/*/langchain_*/*.py | wc -l

# Count TYPE_CHECKING conditional blocks in codebase (excluding tests)
grep -r 'if TYPE_CHECKING:' libs/ --include='*.py' | grep -v test | wc -l

# Run mypy type checking on modules using TYPE_CHECKING imports
python -m mypy libs/partners/ollama/langchain_ollama/ --strict --no-error-summary 2>&1 | grep -c 'Success'
```

**Accept when:**
- All Python modules with optional type dependencies use TYPE_CHECKING conditional imports
- Static type checking (mypy/pyright) passes successfully on all modules using TYPE_CHECKING imports
- Production installations do not require testing frameworks or development-only dependencies
- No NameError exceptions occur in production due to missing TYPE_CHECKING imports

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks MUST block commits with unconditional testing imports. CI/CD pipeline MUST run mypy and pyright type checkers on all Python modules. Code review process MUST flag violations and request changes.
</enforcement>