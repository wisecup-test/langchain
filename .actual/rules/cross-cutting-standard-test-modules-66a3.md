# Standardize Public API Contract Testing with Unit Test Validation: Standard Test Modules

These rules are ALWAYS ACTIVE for all public/external API implementations and partner integrations within the codebase, including all partner integration libraries under libs/partners/*, public API classes in langchain_classic (chains, memory, base classes), chat models, embeddings, client utility modules, router implementations, and base memory abstractions with public interfaces.

### Rules

- **R-API-001** SHOULD: Standard test modules SHOULD include 'test_standard.py' for core contract validation and 'test_imports.py' for public interface verification.

### Verify

```bash
# Count test files in partner integration directories
find libs/partners/*/tests/unit_tests -name 'test_*.py' -type f | wc -l

# Count test functions across partner integrations
grep -r 'def test_' libs/partners/*/tests/unit_tests/ | wc -l

# Execute standard and imports tests
pytest libs/partners/ -v --tb=short -k 'test_standard or test_imports'
```

**Accept when:**
- Each partner integration library contains a 'tests/unit_tests' directory with at least 'test_standard.py' and 'test_imports.py' files
- All public API classes have corresponding unit tests with minimum 80% code coverage
- CI pipeline executes unit tests successfully with zero failures before allowing merge to main branch

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests without required unit tests for public API changes are blocked from merging. CI pipeline fails if unit tests do not pass or coverage falls below threshold. Automated notifications are sent to PR author and reviewers when contract test requirements are not met.
</enforcement>