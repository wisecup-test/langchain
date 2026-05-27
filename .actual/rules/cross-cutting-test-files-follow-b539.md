# Standardize Public API Contract Testing with Unit Test Validation: Test Files Follow

These rules are ALWAYS ACTIVE for all public and external API implementations, partner integrations, and public API classes in langchain_classic that expose interfaces to external consumers.

### Rules

- **R-API-001** SHOULD: Test files SHOULD follow the naming convention 'test_*.py' and be organized in 'unit_tests' or 'tests/unit_tests' directories.

### Verify

```bash
# Count test files following the naming convention
find libs/partners/*/tests/unit_tests -name 'test_*.py' -type f | wc -l

# Count test functions in unit test directories
grep -r 'def test_' libs/partners/*/tests/unit_tests/ | wc -l

# Execute unit tests with standard and import test focus
pytest libs/partners/ -v --tb=short -k 'test_standard or test_imports'
```

**Accept when:**
- Each partner integration library contains a 'tests/unit_tests' directory with at least 'test_standard.py' and 'test_imports.py' files
- All public API classes have corresponding unit tests with minimum 80% code coverage
- CI pipeline executes unit tests successfully with zero failures before allowing merge to main branch
- Test files follow the 'test_*.py' naming convention and are located in designated unit_tests directories

<enforcement>
Claude Code MUST NOT skip or defer verification of test file organization and naming conventions for public API modules. Violations must be flagged during code review and CI pipeline execution.
</enforcement>