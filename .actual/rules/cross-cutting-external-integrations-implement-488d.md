# Standardize External API Integration with Explicit Import Guards: External Integrations Implement

These rules are ALWAYS ACTIVE for all external API integrations, third-party service wrappers, tool implementations, callback handlers, vectorstore implementations, and partner library integrations that depend on optional external dependencies.

### Rules

- **R-EX-001** SHOULD: External API integrations SHOULD implement import guards that allow the module to be imported even when the external dependency is unavailable.
- **R-EX-002** SHOULD: Wrap external imports in try-except blocks at the point of use (class methods or `__init__`), not at module level.
- **R-EX-003** SHOULD: Include the package name and installation command in ImportError messages using the format: `'Could not import {package}. Please install it with `pip install {package}`'`.
- **R-EX-004** MAY: Consider creating a utility function like `require_optional_dependency(package_name, import_name)` to standardize import guard implementation across the codebase.
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
- TYPE_CHECKING imports are used for type hints that would otherwise require optional dependencies at runtime

<enforcement>
Claude Code MUST NOT skip or defer verification of import guards in external integration modules. All new external API integrations MUST be checked for proper import guard implementation before approval.
</enforcement>