# Standardize External API Integration with Explicit Import Guards: Error Messages Missing

These rules are ALWAYS ACTIVE for all tool implementations that wrap external APIs, callback handlers that integrate with external monitoring or logging services, vectorstore implementations that connect to external database services, partner library integrations with third-party services, and any module that imports optional dependencies for external service integration.

### Rules

- **R-EX-001** SHOULD: Error messages for missing dependencies SHOULD include installation instructions (e.g., 'pip install package-name').

### Verify

```bash
# Check for try-except blocks around optional dependency imports
grep -r 'import.*try:' --include='*.py' libs/langchain/langchain_classic/tools/ libs/langchain/langchain_classic/callbacks/ libs/langchain/langchain_classic/vectorstores/

# Check for ImportError messages with installation instructions
grep -r 'ImportError.*install' --include='*.py' libs/

# Identify try-except blocks in Python files
python -c "import ast; import sys; [print(f.name) for f in ast.walk(ast.parse(open(sys.argv[1]).read())) if isinstance(f, ast.Try)]"
```

**Accept when:**
- All external API integration modules contain try-except blocks around optional dependency imports
- ImportError messages include package names and installation instructions
- Modules can be imported successfully even when external dependencies are not installed
- CI tests pass both with and without optional dependencies in the environment

<enforcement>
Claude Code MUST NOT skip or defer verification of import guards and error message completeness in external API integration modules.
</enforcement>