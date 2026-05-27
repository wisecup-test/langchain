# Adopt pytest as Standard Unit Testing Framework: Unit Test Files

These rules are ALWAYS ACTIVE for all unit test files in libs/partners/*/tests/unit_tests/ directories across all partner integration libraries.

### Rules

- **R-PYTEST-001** MUST: Unit test files MUST be organized in a tests/unit_tests/ directory structure within each partner library.
- **R-PYTEST-002** MUST: All test files in unit_tests directories MUST follow the test_*.py naming convention.
- **R-PYTEST-003** MUST: Unit tests MUST be executable via pytest command without requiring alternative test runners.
- **R-PYTEST-004** MUST: No unittest.TestCase or nose-specific imports are permitted in unit test files within the unit_tests directory.
- **R-PYTEST-005** SHOULD: Create conftest.py files at appropriate levels to share fixtures across related tests.
- **R-PYTEST-006** SHOULD: Use pytest markers to categorize tests (e.g., @pytest.mark.unit, @pytest.mark.slow) for selective test execution.
- **R-PYTEST-007** SHOULD: Configure pytest.ini or pyproject.toml with project-specific pytest settings (test paths, markers, plugins).
- **R-PYTEST-008** MAY: Leverage pytest-cov plugin for coverage reporting and pytest-xdist for parallel test execution in CI.

### Verify

```bash
# Count pytest imports in unit test files
grep -r "import pytest" libs/partners/*/tests/unit_tests/ | wc -l

# Find all test files following naming convention
find libs/partners/*/tests/unit_tests -name 'test_*.py' -type f | head -5

# Collect tests via pytest
pytest --collect-only libs/partners/*/tests/unit_tests/ 2>&1 | grep -E '(test session starts|collected)'

# Verify no unittest.TestCase imports in unit test files
grep -r "unittest.TestCase" libs/partners/*/tests/unit_tests/ | wc -l

# Verify no nose-specific imports
grep -r "from nose" libs/partners/*/tests/unit_tests/ | wc -l
```

**Accept when:**
- All test files in unit_tests directories follow the test_*.py naming convention
- pytest successfully discovers and collects tests from all partner library unit test directories
- No unittest.TestCase or nose-specific imports are present in unit test files
- CI pipeline executes tests using pytest command
- grep for unittest.TestCase returns 0 matches
- grep for nose imports returns 0 matches

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for unit test files in the specified scope. Violations must be caught during code review and CI pipeline execution.
</enforcement>