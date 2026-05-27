# Adopt pytest as Standard Unit Testing Framework: Test Files Follow

These rules are ALWAYS ACTIVE for all unit test files in libs/partners/*/tests/unit_tests/ directories and any new test files added to partner integration libraries.

### Rules

- **R-PYTEST-001** MUST: Test files MUST follow the naming convention test_*.py to be discoverable by pytest

### Verify

```bash
# Verify pytest imports are present in test files
grep -r "import pytest" libs/partners/*/tests/unit_tests/ | wc -l

# Find all test files following the test_*.py naming convention
find libs/partners/*/tests/unit_tests -name 'test_*.py' -type f | head -5

# Verify pytest can discover and collect tests
pytest --collect-only libs/partners/*/tests/unit_tests/ 2>&1 | grep -E '(test session starts|collected)'
```

**Accept when:**
- All test files in unit_tests directories follow the test_*.py naming convention
- pytest successfully discovers and collects tests from all partner library unit test directories
- No unittest.TestCase or nose-specific imports are present in unit test files
- CI pipeline executes tests using pytest command

<enforcement>
Claude Code MUST NOT skip or defer verification of test file naming conventions and pytest discoverability.
</enforcement>