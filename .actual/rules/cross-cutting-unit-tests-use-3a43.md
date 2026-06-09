# Adopt pytest as Standard Unit Testing Framework: Unit Tests Use

These rules are ALWAYS ACTIVE for all unit tests in the codebase, particularly those in `libs/partners/*/tests/unit_tests/` directories and any component-specific test files for chat models, embeddings, utilities, and standard interface validation.

### Rules

- **R-PYTEST-001** MUST: All unit tests MUST use pytest as the testing framework.
- **R-PYTEST-002** MUST: All test files MUST follow the `test_*.py` naming convention.
- **R-PYTEST-003** MUST: Unit tests MUST be organized in dedicated test directories following the pattern `libs/partners/{provider}/tests/unit_tests/`.
- **R-PYTEST-004** SHOULD: Create `conftest.py` files at appropriate levels to share fixtures across related tests.
- **R-PYTEST-005** SHOULD: Use pytest markers to categorize tests (e.g., `@pytest.mark.unit`, `@pytest.mark.slow`) for selective test execution.
- **R-PYTEST-006** SHOULD: Configure pytest settings in `pytest.ini` or `pyproject.toml` with project-specific settings (test paths, markers, plugins).
- **R-PYTEST-007** MAY: Leverage pytest-cov plugin for coverage reporting and pytest-xdist for parallel test execution in CI.

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
- No `unittest.TestCase` or nose-specific imports are present in unit test files
- CI pipeline executes tests using pytest command
- pytest is installed as a development dependency in project requirements or pyproject.toml

<enforcement>
Claude Code MUST NOT skip or defer verification. All unit tests must be validated to use pytest before accepting changes to test files or test infrastructure.
</enforcement>