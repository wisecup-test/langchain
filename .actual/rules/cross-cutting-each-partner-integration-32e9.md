# Adopt pytest as Standard Unit Testing Framework: Each Partner Integration

These rules are ALWAYS ACTIVE for all unit tests in `libs/partners/*/tests/unit_tests/` directories across all partner integration libraries.

### Rules

- **R-PYTEST-001** SHOULD: Each partner integration library SHOULD include standard interface tests (`test_standard.py`) and import validation tests (`test_imports.py`).
- **R-PYTEST-002** MUST: All test files in unit_tests directories MUST follow the `test_*.py` naming convention.
- **R-PYTEST-003** MUST: Test files MUST be executable via the `pytest` command with no unittest.TestCase or nose-specific imports present.
- **R-PYTEST-004** SHOULD: pytest.ini or pyproject.toml SHOULD be configured with project-specific pytest settings (test paths, markers, plugins).
- **R-PYTEST-005** SHOULD: conftest.py files SHOULD be created at appropriate levels to share fixtures across related tests.
- **R-PYTEST-006** SHOULD: pytest markers SHOULD be used to categorize tests (e.g., `@pytest.mark.unit`, `@pytest.mark.slow`) for selective test execution.

### Verify

```bash
# Count pytest imports in partner integration unit tests
grep -r "import pytest" libs/partners/*/tests/unit_tests/ | wc -l

# List test files following naming convention
find libs/partners/*/tests/unit_tests -name 'test_*.py' -type f | head -5

# Verify pytest can discover and collect tests
pytest --collect-only libs/partners/*/tests/unit_tests/ 2>&1 | grep -E '(test session starts|collected)'

# Verify no unittest.TestCase imports in unit tests
grep -r "unittest.TestCase" libs/partners/*/tests/unit_tests/ | wc -l

# Verify no nose-specific imports
grep -r "from nose" libs/partners/*/tests/unit_tests/ | wc -l
```

**Accept when:**
- All test files in unit_tests directories follow the `test_*.py` naming convention
- pytest successfully discovers and collects tests from all partner library unit test directories
- No `unittest.TestCase` or nose-specific imports are present in unit test files
- CI pipeline executes tests using `pytest` command
- Standard interface tests (`test_standard.py`) and import validation tests (`test_imports.py`) are present in each partner library

<enforcement>
Claude Code MUST NOT skip or defer verification of pytest adoption across partner integration libraries. All violations MUST be caught during code review and CI pipeline execution.
</enforcement>