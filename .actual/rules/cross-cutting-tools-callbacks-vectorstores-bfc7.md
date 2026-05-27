# Standardize External API Integration with Explicit Import Guards: Tools Callbacks Vectorstores

These rules are ALWAYS ACTIVE for all tool implementations that wrap external APIs, callback handlers that integrate with external monitoring or logging services, vectorstore implementations that connect to external database services, partner library integrations with third-party services, and any module that imports optional dependencies for external service integration.

### Rules

- **R-EX-001** MUST: Tools, callbacks, and vectorstores that depend on external APIs MUST document their optional dependencies in docstrings or class-level comments.
- **R-EX-002** MUST: Wrap external imports in try-except blocks at the point of use (class methods or `__init__`), not at module level.
- **R-EX-003** MUST: Include the package name and installation command in ImportError messages using the format: `'Could not import {package}. Please install it with `pip install {package}`'`.
- **R-EX-004** SHOULD: Create and use a utility function like `require_optional_dependency(package_name, import_name)` to standardize import guard implementation across the codebase.
- **R-EX-005** SHOULD: Document optional dependencies in setup.py/pyproject.toml extras_require section and reference them in error messages.
- **R-EX-006** SHOULD: For type checking, use TYPE_CHECKING imports or string annotations to avoid runtime import requirements.

### Verify

```bash
# Check for try-except blocks around external imports in integration modules
grep -r 'import.*try:' --include='*.py' libs/langchain/langchain_classic/tools/ libs/langchain/langchain_classic/callbacks/ libs/langchain/langchain_classic/vectorstores/

# Verify ImportError messages include installation instructions
grep -r 'ImportError.*install' --include='*.py' libs/

# Identify try-except blocks in Python files
python -c "import ast; import sys; [print(f.name) for f in ast.walk(ast.parse(open(sys.argv[1]).read())) if isinstance(f, ast.Try)]"
```

**Accept when:**
- All external API integration modules contain try-except blocks around optional dependency imports
- ImportError messages include package names and installation instructions
- Modules can be imported successfully even when external dependencies are not installed
- CI tests pass both with and without optional dependencies in the environment
- Optional dependencies are documented in docstrings or class-level comments
- Type checking uses TYPE_CHECKING imports or string annotations where applicable

<enforcement>
Claude Code MUST NOT skip or defer verification of import guards in external API integration modules. All violations MUST be flagged during code review and linting checks MUST pass before merge.
</enforcement>