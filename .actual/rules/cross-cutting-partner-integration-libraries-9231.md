# Standardize Public API Contract Testing with Unit Test Validation: Partner Integration Libraries

These rules are ALWAYS ACTIVE for all public/external API implementations and partner integrations within the codebase, including partner integration libraries (e.g., OpenAI, Ollama, Nomic), public API classes in langchain_classic (chains, memory, base classes), chat models, embeddings, client utility modules, router implementations, and base memory abstractions with public interfaces.

### Rules

- **R-PARTNER-001** MUST: Partner integration libraries (e.g., OpenAI, Ollama, Nomic) implement standard unit tests covering embeddings, chat models, and client utilities.
- **R-PARTNER-002** MUST: All public API classes have corresponding unit tests with minimum 80% code coverage.
- **R-PARTNER-003** MUST: Each partner integration library contains a 'tests/unit_tests' directory with at least 'test_standard.py' and 'test_imports.py' files.
- **R-PARTNER-004** MUST: Unit tests validate essential contracts including valid inputs with expected outputs, invalid inputs with expected errors, edge cases, and backward compatibility scenarios.
- **R-PARTNER-005** MUST: CI pipeline executes unit tests successfully with zero failures before allowing merge to main branch.

### Verify

```bash
# Count unit test files in partner libraries
find libs/partners/*/tests/unit_tests -name 'test_*.py' -type f | wc -l

# Count test functions across partner libraries
grep -r 'def test_' libs/partners/*/tests/unit_tests/ | wc -l

# Execute standard and import tests
pytest libs/partners/ -v --tb=short -k 'test_standard or test_imports'
```

**Accept when:**
- Each partner integration library contains a 'tests/unit_tests' directory with at least 'test_standard.py' and 'test_imports.py' files
- All public API classes have corresponding unit tests with minimum 80% code coverage
- CI pipeline executes unit tests successfully with zero failures before allowing merge to main branch
- Unit tests cover valid inputs with expected outputs, invalid inputs with expected errors, edge cases, and backward compatibility scenarios

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests without required unit tests for public API changes are blocked from merging. CI pipeline fails if unit tests do not pass or coverage falls below threshold. Automated notifications are sent to PR author and reviewers when contract test requirements are not met.
</enforcement>