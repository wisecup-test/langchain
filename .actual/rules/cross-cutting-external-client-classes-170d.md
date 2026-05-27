# Standardize External API Integration with Explicit Import Guards: External Client Classes

These rules are ALWAYS ACTIVE for all external API client classes, tool implementations wrapping external APIs, callback handlers integrating with external services, vectorstore implementations connecting to external databases, and partner library integrations with third-party services.

### Rules

- **R-EX-001** MUST: External API client classes MUST defer dependency imports until class instantiation or method invocation, not at module load time.

### Verify

```bash
# Check for try-except blocks around optional dependency imports in external integration modules
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

<enforcement>
Claude Code MUST NOT skip or defer verification of import guard patterns in external API client classes. Violations block PR merge until import guards are added.
</enforcement>