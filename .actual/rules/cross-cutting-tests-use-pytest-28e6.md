# Adopt pytest as Standard Unit Testing Framework: Tests Use Pytest

These rules are ALWAYS ACTIVE for all unit tests in `libs/partners/*/tests/unit_tests/` directories and any new test files added to partner integration libraries.

### Rules

- **R-PYTEST-001** MUST: All unit tests in partner libraries SHALL use pytest as the testing framework.
- **R-PYTEST-002** MUST: Test files SHALL follow the `test_*.py` naming convention for pytest discovery.
- **R-PYTEST-003** MUST: Test files SHALL NOT import `unittest.TestCase` or nose-specific testing frameworks in unit test directories.
- **R-PYTEST-004** MAY: Tests MAY use pytest fixtures, parametrization, and plugins to enhance test capabilities.
- **R-PYTEST-005** SHOULD: Tests SHOULD use pytest markers (e.g., `@pytest.mark.unit`, `@pytest.mark.slow`) to categorize and enable selective test execution.
- **R-PYTEST-006** SHOULD: Projects SHOULD create `conftest.py` files at appropriate levels to share fixtures across related tests.
- **R-PYTEST-007** SHOULD: Projects SHOULD configure pytest settings in `pytest.ini` or `pyproject.toml` with test paths, markers, and plugins.

### Verify

```bash
# Verify pytest imports are present in unit tests
grep -r "import pytest" libs/partners/*/tests/unit_tests/ | wc -l

# Verify test file naming convention
find libs/partners/*/tests/unit_tests -name 'test_*.py' -type f | head -5

# Verify pytest can discover and collect tests
pytest --collect-only libs/partners/*/tests/unit_tests/ 2>&1 | grep -E '(test session starts|collected)'

# Verify no unittest.TestCase imports in unit tests
grep -r "unittest.TestCase" libs/partners/*/tests/unit_tests/ | wc -l

# Verify no nose-specific imports in unit tests
grep -r "from nose" libs/partners/*/tests/unit_tests/ | wc -l
```

**Accept when:**
- All test files in unit_tests directories follow the `test_*.py` naming convention
- pytest successfully discovers and collects tests from all partner library unit test directories
- No `unittest.TestCase` or nose-specific imports are present in unit test files
- CI pipeline executes tests using the `pytest` command
- grep for pytest imports returns a count greater than zero for active test directories

<enforcement>
Claude Code MUST NOT skip or defer verification. All new test files in partner libraries MUST be verified to use pytest before approval. CI pipeline MUST fail if tests are not executable via pytest.
</enforcement>