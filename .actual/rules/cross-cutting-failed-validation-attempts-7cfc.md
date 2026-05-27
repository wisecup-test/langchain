# Enforce Input Validation for External Data Sources in Security-Critical Components: Failed Validation Attempts

These rules are ALWAYS ACTIVE for all chat model integrations, embedding model interfaces, prompt selector and template processing components, callback handlers, configuration parsers, and test utilities that handle external data sources, API integrations, and user-provided inputs in security-critical contexts.

### Rules

- **R-VALIDATION-001** SHOULD: Failed validation attempts SHOULD be logged with sufficient detail for security monitoring while avoiding exposure of sensitive data.

### Verify

```bash
# Count type checking and validation patterns in integration code
grep -r "isinstance\|type(" libs/partners/*/langchain_*/chat_models.py libs/partners/*/langchain_*/embeddings.py | wc -l

# Count validation and schema validation usage
grep -r "validate\|ValidationError\|pydantic" libs/partners/*/langchain_*/*.py | wc -l

# Count validation-related test cases
python -m pytest tests/ -k "test_invalid_input or test_validation" --collect-only | grep "<Function" | wc -l
```

**Accept when:**
- All integration points with external providers include explicit type checking or schema validation before processing responses
- At least 80% of integration components use declarative validation (Pydantic, dataclasses with validators, or similar)
- Test suites include negative test cases for invalid inputs in all security-critical integration points
- Failed validation attempts are logged with sufficient context for security monitoring, with sensitive data sanitized

<enforcement>
Claude Code MUST NOT skip or defer verification. All integration points handling external data must include validation logging that captures sufficient detail for security monitoring without exposing sensitive information. Code review and CI pipeline checks are mandatory before merge.
</enforcement>