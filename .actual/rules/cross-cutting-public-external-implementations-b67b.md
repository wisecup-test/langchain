# Standardize Public API Contract Testing with Unit Test Validation: Public External Implementations

These rules are ALWAYS ACTIVE for all public/external API implementations, partner integrations, and public API classes intended for direct consumption by external developers.

### Rules

- **R-PUB-001** MUST: All public/external API implementations MUST include comprehensive unit tests that validate the API contract.

### Verify

```bash
# Count unit test files in partner libraries
find libs/partners/*/tests/unit_tests -name 'test_*.py' -type f | wc -l

# Count test functions across partner integrations
grep -r 'def test_' libs/partners/*/tests/unit_tests/ | wc -l

# Execute contract and import tests
pytest libs/partners/ -v --tb=short -k 'test_standard or test_imports'
```

**Accept when:**
- Each partner integration library contains a `tests/unit_tests` directory with at least `test_standard.py` and `test_imports.py` files
- All public API classes have corresponding unit tests with minimum 80% code coverage
- CI pipeline executes unit tests successfully with zero failures before allowing merge to main branch
- Unit tests cover valid inputs with expected outputs, invalid inputs with expected errors, edge cases, and backward compatibility scenarios

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests without required unit tests for public API changes are blocked from merging. CI pipeline fails if unit tests do not pass or coverage falls below threshold.
</enforcement>