# Adopt Type Checking Conditional Imports for Testing Dependencies: Type Checking Blocks

These rules are ALWAYS ACTIVE for all Python modules that require type hints from testing or optional dependencies, including all modules in partner integration libraries (libs/partners/*), core langchain modules using optional dependencies for type hints, and any module importing from pytest, unittest, or other testing frameworks solely for type annotations.

### Rules

- **R-TYPE-001** SHOULD: TYPE_CHECKING blocks SHOULD be placed immediately after standard library imports and before third-party imports.
- **R-TYPE-002** MUST: Always import TYPE_CHECKING at the top of the module: `from typing import TYPE_CHECKING`.
- **R-TYPE-003** SHOULD: When using TYPE_CHECKING imports in function signatures, use string literals for type annotations: `def foo(param: 'OptionalType') -> None:`.
- **R-TYPE-004** SHOULD: Group related TYPE_CHECKING imports together and add comments explaining why they're conditional.
- **R-TYPE-005** MUST NOT: Use TYPE_CHECKING imports directly in runtime code without string annotations, as this will cause NameError exceptions in production.
- **R-TYPE-006** MUST NOT: Include testing frameworks or development-only dependencies in production package requirements.

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
- TYPE_CHECKING blocks are positioned after standard library imports and before third-party imports
- String annotations are used for all TYPE_CHECKING-imported types in runtime function signatures

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks MUST block commits with unconditional testing imports. CI/CD pipeline MUST run mypy and pyright type checkers on all Python modules. Code review MUST verify TYPE_CHECKING usage for optional dependencies. Runtime monitoring MUST alert on NameError exceptions related to missing type imports.
</enforcement>