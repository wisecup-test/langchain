# Adopt pytest as Standard Unit Testing Framework: Unit Tests Organized

These rules are ALWAYS ACTIVE for all unit tests in `libs/partners/*/tests/unit_tests/` directories and component-specific test files across partner integration libraries.

### Rules

- **R-PYTEST-001** SHOULD: Unit tests SHOULD be organized by component type (e.g., chat_models, embeddings, imports) in subdirectories within `libs/partners/{provider}/tests/unit_tests/`.
- **R-PYTEST-002** MUST: All test files MUST follow the `test_*.py` naming convention.
- **R-PYTEST-003** MUST: pytest MUST be used as the test runner for all unit tests in partner libraries.
- **R-PYTEST-004** SHOULD: conftest.py files SHOULD be created at appropriate levels to share fixtures across related tests.
- **R-PYTEST-005** SHOULD: pytest markers (e.g., `@pytest.mark.unit`, `@pytest.mark.slow`) SHOULD be used to categorize tests for selective execution.
- **R-PYTEST-006** MUST: No unittest.TestCase or nose-specific imports MUST be present in unit test files.
- **R-PYTEST-007** SHOULD: pytest.ini or pyproject.toml SHOULD be configured with project-specific pytest settings (test paths, markers, plugins).
- **R-PYTEST-008** SHOULD: pytest-cov plugin SHOULD be leveraged for coverage reporting and pytest-xdist for parallel test execution in CI.

### Verify

```bash
# Count pytest imports in unit test directories
grep -r "import pytest" libs/partners/*/tests/unit_tests/ | wc -l

# List test files following naming convention
find libs/partners/*/tests/unit_tests -name 'test_*.py' -type f | head -5

# Collect tests via pytest
pytest --collect-only libs/partners/*/tests/unit_tests/ 2>&1 | grep -E '(test session starts|collected)'

# Verify no unittest.TestCase imports in unit tests
grep -r "unittest.TestCase" libs/partners/*/tests/unit_tests/ | wc -l

# Verify no nose-specific imports
grep -r "from nose" libs/partners/*/tests/unit_tests/ | wc -l
```

**Accept when:**
- All test files in unit_tests directories follow the `test_*.py` naming convention
- pytest successfully discovers and collects tests from all partner library unit test directories
- No unittest.TestCase or nose-specific imports are present in unit test files
- CI pipeline executes tests using pytest command
- conftest.py files exist at appropriate directory levels for fixture sharing
- pytest markers are applied to categorize tests

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All unit tests in partner libraries MUST comply with pytest standards and organizational requirements before code review approval.
</enforcement>