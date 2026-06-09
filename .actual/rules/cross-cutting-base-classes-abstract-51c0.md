# Standardize Public API Contract Testing with Unit Test Validation: Base Classes Abstract

These rules are ALWAYS ACTIVE for all public and external API implementations, partner integrations, base classes, and abstract interfaces exposed as public APIs within the codebase.

### Rules

- **R-API-001** MUST: Base classes and abstract interfaces exposed as public APIs MUST have unit tests validating their contract requirements.

### Verify

```bash
# Count unit test files in partner libraries
find libs/partners/*/tests/unit_tests -name 'test_*.py' -type f | wc -l

# Count test functions across partner libraries
grep -r 'def test_' libs/partners/*/tests/unit_tests/ | wc -l

# Execute standard and import contract tests
pytest libs/partners/ -v --tb=short -k 'test_standard or test_imports'
```

**Accept when:**
- Each partner integration library contains a 'tests/unit_tests' directory with at least 'test_standard.py' and 'test_imports.py' files
- All public API classes have corresponding unit tests with minimum 80% code coverage
- CI pipeline executes unit tests successfully with zero failures before allowing merge to main branch
- Unit tests cover valid inputs with expected outputs, invalid inputs with expected errors, edge cases, and backward compatibility scenarios

<enforcement>
Claude Code MUST NOT skip or defer verification of R-API-001. Pull requests without required unit tests for public API changes are blocked from merging. CI pipeline fails if unit tests do not pass or coverage falls below threshold.
</enforcement>