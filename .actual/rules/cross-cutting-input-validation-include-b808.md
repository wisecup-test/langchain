# Enforce Input Validation for External Data Sources in Security-Critical Components: Input Validation Include

These rules are ALWAYS ACTIVE for all chat model integrations, embedding model interfaces, prompt selector and template processing components, callback handlers, configuration parsers, and test utilities that handle external data sources, API integrations, and user-provided inputs in security-critical contexts.

### Rules

- **R-VALIDATION-001** SHOULD: Input validation SHOULD include bounds checking for numeric values, length limits for strings, and whitelist validation for enumerated types.

### Verify

```bash
# Count type checking patterns in integration files
grep -r "isinstance\|type(" libs/partners/*/langchain_*/chat_models.py libs/partners/*/langchain_*/embeddings.py | wc -l

# Count validation and schema validation usage
grep -r "validate\|ValidationError\|pydantic" libs/partners/*/langchain_*/*.py | wc -l

# Count validation test cases
python -m pytest tests/ -k "test_invalid_input or test_validation" --collect-only | grep "<Function" | wc -l
```

**Accept when:**
- All integration points with external providers include explicit type checking or schema validation before processing responses
- At least 80% of integration components use declarative validation (Pydantic, dataclasses with validators, or similar)
- Test suites include negative test cases for invalid inputs in all security-critical integration points

<enforcement>
Claude Code MUST NOT skip or defer verification. Validation at integration boundaries is mandatory for all external data sources. Violations must be caught during code review and CI pipeline checks.
</enforcement>